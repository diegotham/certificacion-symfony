Los **voters** son simplemente **sentencias condicionales**. Para emplear voters hay que entender cómo los utiliza **Symfony**. Todos los voters se llaman cada vez que se llama al método _isGranted()_ en el **authorization checker** de Symfony (por ejemplo el service _security.authorization_checker_). Cada uno de los voters decide si el usuario acual debería tener acceso a algún resource.

Finalmente, Symfony recoge las respuestas de todos los voters y toma una decisión final (permitir o denegar el acceso a un resource) en función de la estrategia definida en la aplicación, que puede ser: _affirmative_, _consensus_ o _unanimous_.

### La interface voter

Un voter personalizado necesita implementar [VoterInterface](http://api.symfony.com/3.0/Symfony/Component/Security/Core/Authorization/Voter/VoterInterface.html) o extender [Voter](http://api.symfony.com/3.0/Symfony/Component/Security/Core/Authorization/Voter/Voter.html). Con esta segunda es más fácil crear un voter.

```
abstract class Voter implements VoterInterface
{
    abstract protected function supports($attribute, $subject);
    abstract protected function voteOnAttribute($attribute, $subject, TokenInterface $token);
}
```

### Comprobar el acceso en un controller

Por ejemplo tenemos un objeto **Post** y queremos decidir si el usuario actual puede editar (_edit_) o ver (_view_) el objeto. En el **controller** podemos controlar el acceso de la siguiente forma:

```
// src/AppBundle/Controller/PostController.php
// ...

class PostController extends Controller
{
    /**
     * @Route("/posts/{id}", name="post_show")
     */
    public function showAction($id)
    {
        // obtiene un objeto Post - e.g. lo consulta
        $post = ...;

        // comprobar el acceso para "view": llama a todos los voters
        $this->denyAccessUnlessGranted('view', $post);

        // ...
    }

    /**
     * @Route("/posts/{id}/edit", name="post_edit")
     */
    public function editAction($id)
    {
        // obtiene un objeto Post object - e.g. lo consulta
        $post = ...;

        // comprobar el acceso para "edit": llama a todos los voters
        $this->denyAccessUnlessGranted('edit', $post);

        // ...
    }
}
```

El método _denyAccessUnlessGranted()_ (y también su simplificación _isGranted()_) llama al sistema de voters. Ahora mismo no hay voters que puedan juzgar si el usuario puede editar o ver el Post, pero puedes crear tu propio voter que lo decide empleando la lógica que quieras.

### Crear un voter personalizado

Supongamos que la lógica de decidir si un usuario puede ver o editar un objeto **Post** es la siguiente: un usuario puede siempre editar y ver un Post que haya creado, y si un Post es marcado como "_public_", cualquiera puede verlo. Un voter para esta situación podría ser como sigue:

```
// src/AppBundle/Security/PostVoter.php
namespace AppBundle\Security;

use AppBundle\Entity\Post;
use AppBundle\Entity\User;
use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;
use Symfony\Component\Security\Core\Authorization\Voter\Voter;

class PostVoter extends Voter
{
    // en estos strings puedes poner lo que quieras
    const VIEW = 'view';
    const EDIT = 'edit';

    protected function supports($attribute, $subject)
    {
        // si el atributo no es uno de los que soportamos, devolver false
        if (!in_array($attribute, array(self::VIEW, self::EDIT))) {
            return false;
        }

        // sólo votar en objetos Post dentro de este voter
        if (!$subject instanceof Post) {
            return false;
        }

        return true;
    }

    protected function voteOnAttribute($attribute, $subject, TokenInterface $token)
    {
        $user = $token->getUser();

        if (!$user instanceof User) {
            // el usuario debe estar logeado; sino, denegar el acceso
            return false;
        }

        // $subject es un objeto Post, gracias al método supports
        /** @var Post $post */
        $post = $subject;

        switch($attribute) {
            case self::VIEW:
                return $this->canView($post, $user);
            case self::EDIT:
                return $this->canEdit($post, $user);
        }

        throw new \LogicException('Este código no debería ser visto');
    }

    private function canView(Post $post, User $user)
    {
        // si pueden editar, pueden ver
        if ($this->canEdit($post, $user)) {
            return true;
        }

        // el objeto Post podría tener, por ejemplo, un método isPrivate()
        // que comprueba la propiedad booleana $private
        return !$post->isPrivate();
    }

    private function canEdit(Post $post, User $user)
    {
        // esto asume que el objeto tiene un método getOwner()
        // para obtener la entidad del usuario que posee este objeto
        return $user === $post->getOwner();
    }
}
```

Esto es lo que se espera de los dos métodos abstractos:

*   **Voter::supports($attribute, $subject)**. Cuando se llama a _isGranted()_ (o _denyAccessUnlessGranted()_), el primer argumento se pasa aquí como **$attribute** (por ejemplo ROLE_USER, edit) y el segundo argumento (si existe) se pasa como **$subject** (por ejemplo null, o un objeto Post). Nuestra tarea es determinar si el voter debería votar en la combinación attribute/subject. Si devuelve true se llamará a _voteOnAttribute()_. Sino, el voter termina: algún otro voter tendría que procesarlo. En este ejemplo devolvemos true si el atributo es _view_ o _edit_ y si el objeto es una instancia de Post.
*   **voteOnAttribute($attribute, $subject, TokenInterface $token)**. Si devuelves true desde _supports()_, se llama a este método. Nuestra tarea ahora es simple: devolver **true** para permitir el acceso y **false** para denegarlo. El $token puede usarse para encontrar el objeto user actual (si existe). En este ejemplo, toda la business logic se incluye para determinar el acceso.

### Configurar el voter

Para inyectar el voter en el **security layer** debes declararlo como service y ponerle el _tag_ **security.voter**:

```
# app/config/services.yml
services:
    app.post_voter:
        class: AppBundle\Security\PostVoter
        tags:
            - { name: security.voter }
        # pequeña mejora de rendimiento
        public: false
```

Ahora cuando llamemos a isGranted() con view o edit en un objeto Post, el voter se ejecutará y puedes controlar el acceso.

### Comprobar los roles en un voter

Si queremos llamar a _isGranted()_ desde dentro de un voter (por ejemplo queremos comprobar si el usuario actual tiene role **ROLE_SUPER_ADMIN**). Esto se puede hacer inyectando el AccessDecisionManager en el voter. Podemos emplear esto para, por ejemplo, permitir siempre el acceso a un usuario con el role ROLE_SUPER_ADMIN:

```
// src/AppBundle/Security/PostVoter.php

// ...
use Symfony\Component\Security\Core\Authorization\AccessDecisionManagerInterface;

class PostVoter extends Voter
{
    // ...

    private $decisionManager;

    public function __construct(AccessDecisionManagerInterface $decisionManager)
    {
        $this->decisionManager = $decisionManager;
    }

    protected function voteOnAttribute($attribute, $subject, TokenInterface $token)
    {
        // ...

        // ROLE_SUPER_ADMIN can do anything! The power!
        if ($this->decisionManager->decide($token, array('ROLE_SUPER_ADMIN'))) {
            return true;
        }

        // ... toda la lógica del voter normal
    }
}
```

Después, actualiza _services.yml_ para inyectar el service **security.access.decision_manager**:

```
# app/config/services.yml
services:
    app.post_voter:
        class: AppBundle\Security\PostVoter
        arguments: ['@security.access.decision_manager']
        public: false
        tags:
            - { name: security.voter }
```

Eso es todo. Llamar a _decide()_ en el **AccessDecisionManager** es esencialmente lo mismo que llamar a _isGranted()_ desde un controller u otros sitios.

El _security.access.decision_manager_ es **private**. Esto significa que no puedes acceder directamente desde un controller, sólo puedes inyectarlo en otros services. No importa, puedes emplear _security.authorization_checker_ en su lugar en todos los casos excepto para voters.

### Cambiar la estrategia de decisión de acceso

Normalmente sólo un **voter** votará en cada ocasión (el resto se abstendrán, lo que significa que devolverán **false** desde _supports()_). Pero también podrías hacer que múltiples voters voten para una acción y objeto. Por ejemplo, suponemos que tenemos otro voter que comprueba si el usuario es un miembro del sitio y un segundo que comprueba si el usuario es mayor de 18. 

Para manejar estos casos, el **access decision manager** utiliza una **access decision strategy**. Puedes configurar esto según tus necesidades. Hay tres estrategias disponibles:

*   _affirmative_ (por defecto). Otorga acceso tan pronto como haya un voter que permita acceso.
*   _consensus_. Otorga acceso si hay más voters garantizando acceso que denegándolo.
*   _unanimous_. Sólo otorga acceso una vez que todos los voters garantizan acceso.

En el escenario anterior, ambos voters deberían permitir el acceso para que el usuario pueda leer el post. En este caso, la estrategia por defecto ya no es válida y se debería utilizar _unanimous_. Se puede establecer esto en la configuración de seguridad:

```
# app/config/security.yml
security:
    access_decision_manager:
        strategy: unanimous
```