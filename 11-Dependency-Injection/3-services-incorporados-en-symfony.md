**Symfony** y los **bundles de terceros** configuran y obtienen sus services a través del **container**, por lo que puedes acceder a ellos o usarlos en tus propios services. Symfony por defecto no requiere que los **controllers** se definan como services, e inyecta el **service container** en los controllers. Por ejemplo, para manejar la información de almacenamiento de la **sesión de un usuario**, Symfony proporciona un service _session_, al que puedes acceder desde un controller normal:

```
public function indexAction($bar)
{
    $session = $this->get('session');
    $session->set('foo', $bar);

    // ...
}
```

En Symfony se emplean continuamente services proporcionados por el core de Symfony o por bundles de terceros para hacer tareas como renderizar **templates** (_templating_), enviar **emails** (_mailer_), o acceder a información del **request** (_request_).

Puedes también emplear estos services dentro de services que hayas creado para tu **aplicación**. Vamos a utilizar el Symfony _mailer service_ para una clase **NewsletterManager**, y pasaremos el templating service a la misma clase para que pueda generar el contenido del email a través de una template:

```
// src/AppBundle/Newsletter/NewsletterManager.php
namespace AppBundle\Newsletter;

use Symfony\Component\Templating\EngineInterface;

class NewsletterManager
{
    protected $mailer;

    protected $templating;

    public function __construct(\Swift_Mailer $mailer, EngineInterface $templating)
    {
        $this->mailer = $mailer;
        $this->templating = $templating;
    }

    // ...
}
```

Ahora podemos introducir las **dependencias** en el service:

```
# app/config/services.yml
services:
    app.newsletter_manager:
        class:     AppBundle\Newsletter\NewsletterManager
        arguments: ['@mailer', '@templating']
```

El service _app.newsletter_manager_ ahora tiene acceso a los core services _mailer_ y _templating_. 

Podemos mostrar un listado de todos los services (y sus clases) que están registrados en el container con la **consola**, con el siguiente comando:

```
php bin/console debug:container
```

Por defecto sólo se muestran _public services_, pero puedes también ver _private services_ con

```

php bin/console debug:container --show-private
```

### Listado completo de services disponibles en Symfony 3

1.  **annotation_reader** Doctrine\Common\Annotations\CachedReader
2.  **assets.context** Symfony\Component\Asset\Context\RequestStackContext
3.  **assets.packages** Symfony\Component\Asset\Packages
4.  **cache_clearer** Symfony\Component\HttpKernel\CacheClearer\ChainCacheClearer
5.  **cache_warmer** Symfony\Component\HttpKernel\CacheWarmer\CacheWarmerAggregate
6.  **config_cache_factory** Symfony\Component\Config\ResourceCheckerConfigCacheFactory
7.  **console.command.sensiolabs_security_command_securitycheckercommand** alias for "sensio_distribution.security_checker.command"
8.  **data_collector.dump** Symfony\Component\HttpKernel\DataCollector\DumpDataCollector
9.  **data_collector.form** Symfony\Component\Form\Extension\DataCollector\FormDataCollector
10.  **data_collector.form.extractor** Symfony\Component\Form\Extension\DataCollector\FormDataExtractor
11.  **data_collector.request** Symfony\Component\HttpKernel\DataCollector\RequestDataCollector
12.  **data_collector.router** Symfony\Bundle\FrameworkBundle\DataCollector\RouterDataCollector
13.  **database_connection** alias for "doctrine.dbal.default_connection"
14.  **debug.controller_resolver** Symfony\Component\HttpKernel\Controller\TraceableControllerResolver
15.  **debug.debug_handlers_listener** Symfony\Component\HttpKernel\EventListener\DebugHandlersListener
16.  **debug.dump_listener** Symfony\Component\HttpKernel\EventListener\DumpListener
17.  **debug.event_dispatcher** Symfony\Component\HttpKernel\Debug\TraceableEventDispatcher
18.  **debug.stopwatch** Symfony\Component\Stopwatch\Stopwatch doctrine Doctrine\Bundle\DoctrineBundle\Registry
19.  **doctrine.dbal.connection_factory** Doctrine\Bundle\DoctrineBundle\ConnectionFactory
20.  **doctrine.dbal.default_connection** Doctrine\DBAL\Connection
21.  **doctrine.orm.default_entity_listener_resolver** Doctrine\ORM\Mapping\DefaultEntityListenerResolver
22.  **doctrine.orm.default_entity_manager** Doctrine\ORM\EntityManager
23.  **doctrine.orm.default_listeners.attach_entity_listeners** Doctrine\ORM\Tools\AttachEntityListenersListener
24.  **doctrine.orm.default_manager_configurator** Doctrine\Bundle\DoctrineBundle\ManagerConfigurator
25.  **doctrine.orm.default_metadata_cache** alias for "doctrine_cache.providers.doctrine.orm.default_metadata_cache"
26.  **doctrine.orm.default_query_cache** alias for "doctrine_cache.providers.doctrine.orm.default_query_cache"
27.  **doctrine.orm.default_result_cache** alias for "doctrine_cache.providers.doctrine.orm.default_result_cache"
28.  **doctrine.orm.entity_manager** alias for "doctrine.orm.default_entity_manager"
29.  **doctrine.orm.validator.unique** Symfony\Bridge\Doctrine\Validator\Constraints\UniqueEntityValidator
30.  **doctrine.orm.validator_initializer** Symfony\Bridge\Doctrine\Validator\DoctrineInitializer
31.  **doctrine_cache.providers.doctrine.orm.default_metadata_cache** Doctrine\Common\Cache\ArrayCache
32.  **doctrine_cache.providers.doctrine.orm.default_query_cache** Doctrine\Common\Cache\ArrayCache
33.  **doctrine_cache.providers.doctrine.orm.default_result_cache** Doctrine\Common\Cache\ArrayCache
34.  **event_dispatcher** alias for "debug.event_dispatcher"
35.  **file_locator** Symfony\Component\HttpKernel\Config\FileLocator
36.  **filesystem** Symfony\Component\Filesystem\Filesystem
37.  **form.factory** Symfony\Component\Form\FormFactory
38.  **form.registry** Symfony\Component\Form\FormRegistry
39.  **form.resolved_type_factory** Symfony\Component\Form\Extension\DataCollector\Proxy\ResolvedTypeFactoryDataCollectorProxy
40.  **form.type.birthday** Symfony\Component\Form\Extension\Core\Type\BirthdayType
41.  **form.type.button** Symfony\Component\Form\Extension\Core\Type\ButtonType
42.  **form.type.checkbox** Symfony\Component\Form\Extension\Core\Type\CheckboxType
43.  **form.type.choice** Symfony\Component\Form\Extension\Core\Type\ChoiceType
44.  **form.type.collection** Symfony\Component\Form\Extension\Core\Type\CollectionType
45.  **form.type.country** Symfony\Component\Form\Extension\Core\Type\CountryType
46.  **form.type.currency** Symfony\Component\Form\Extension\Core\Type\CurrencyType
47.  **form.type.date** Symfony\Component\Form\Extension\Core\Type\DateType
48.  **form.type.datetime** Symfony\Component\Form\Extension\Core\Type\DateTimeType
49.  **form.type.email** Symfony\Component\Form\Extension\Core\Type\EmailType
50.  **form.type.entity** Symfony\Bridge\Doctrine\Form\Type\EntityType
51.  **form.type.file** Symfony\Component\Form\Extension\Core\Type\FileType
52.  **form.type.form** Symfony\Component\Form\Extension\Core\Type\FormType
53.  **form.type.hidden** Symfony\Component\Form\Extension\Core\Type\HiddenType
54.  **form.type.integer** Symfony\Component\Form\Extension\Core\Type\IntegerType
55.  **form.type.language** Symfony\Component\Form\Extension\Core\Type\LanguageType
56.  **form.type.locale** Symfony\Component\Form\Extension\Core\Type\LocaleType
57.  **form.type.money** Symfony\Component\Form\Extension\Core\Type\MoneyType
58.  **form.type.number** Symfony\Component\Form\Extension\Core\Type\NumberType
59.  **form.type.password** Symfony\Component\Form\Extension\Core\Type\PasswordType
60.  **form.type.percent** Symfony\Component\Form\Extension\Core\Type\PercentType
61.  **form.type.radio** Symfony\Component\Form\Extension\Core\Type\RadioType
62.  **form.type.range** Symfony\Component\Form\Extension\Core\Type\RangeType
63.  **form.type.repeated** Symfony\Component\Form\Extension\Core\Type\RepeatedType
64.  **form.type.reset** Symfony\Component\Form\Extension\Core\Type\ResetType
65.  **form.type.search** Symfony\Component\Form\Extension\Core\Type\SearchType
66.  **form.type.submit** Symfony\Component\Form\Extension\Core\Type\SubmitType
67.  **form.type.text** Symfony\Component\Form\Extension\Core\Type\TextType
68.  **form.type.textarea** Symfony\Component\Form\Extension\Core\Type\TextareaType
69.  **form.type.time** Symfony\Component\Form\Extension\Core\Type\TimeType
70.  **form.type.timezone** Symfony\Component\Form\Extension\Core\Type\TimezoneType
71.  **form.type.url** Symfony\Component\Form\Extension\Core\Type\UrlType
72.  **form.type_extension.csrf** Symfony\Component\Form\Extension\Csrf\Type\FormTypeCsrfExtension
73.  **form.type_extension.form.data_collector** Symfony\Component\Form\Extension\DataCollector\Type\DataCollectorTypeExtension
74.  **form.type_extension.form.http_foundation** Symfony\Component\Form\Extension\HttpFoundation\Type\FormTypeHttpFoundationExtension
75.  **form.type_extension.form.validator** Symfony\Component\Form\Extension\Validator\Type\FormTypeValidatorExtension
76.  **form.type_extension.repeated.validator** Symfony\Component\Form\Extension\Validator\Type\RepeatedTypeValidatorExtension
77.  **form.type_extension.submit.validator** Symfony\Component\Form\Extension\Validator\Type\SubmitTypeValidatorExtension
78.  **form.type_guesser.doctrine** Symfony\Bridge\Doctrine\Form\DoctrineOrmTypeGuesser
79.  **form.type_guesser.validator** Symfony\Component\Form\Extension\Validator\ValidatorTypeGuesser
80.  **fragment.handler** Symfony\Component\HttpKernel\DependencyInjection\LazyLoadingFragmentHandler
81.  **fragment.listener** Symfony\Component\HttpKernel\EventListener\FragmentListener
82.  **fragment.renderer.esi** Symfony\Component\HttpKernel\Fragment\EsiFragmentRenderer
83.  **fragment.renderer.hinclude** Symfony\Component\HttpKernel\Fragment\HIncludeFragmentRenderer
84.  **fragment.renderer.inline** Symfony\Component\HttpKernel\Fragment\InlineFragmentRenderer
85.  **fragment.renderer.ssi** Symfony\Component\HttpKernel\Fragment\SsiFragmentRenderer
86.  **http_kernel** Symfony\Component\HttpKernel\HttpKernel
87.  **kernel**
88.  **kernel.class_cache.cache_warmer** Symfony\Bundle\FrameworkBundle\CacheWarmer\ClassCacheCacheWarmer
89.  **locale_listener** Symfony\Component\HttpKernel\EventListener\LocaleListener logger Symfony\Bridge\Monolog\Logger
90.  **mailer** alias for "swiftmailer.mailer.default"
91.  **monolog.handler.console** Symfony\Bridge\Monolog\Handler\ConsoleHandler
92.  **monolog.handler.debug** Symfony\Bridge\Monolog\Handler\DebugHandler
93.  **monolog.handler.main** Monolog\Handler\StreamHandler
94.  **monolog.logger.doctrine** Symfony\Bridge\Monolog\Logger
95.  **monolog.logger.event** Symfony\Bridge\Monolog\Logger
96.  **monolog.logger.php** Symfony\Bridge\Monolog\Logger
97.  **monolog.logger.profiler** Symfony\Bridge\Monolog\Logger
98.  **monolog.logger.request** Symfony\Bridge\Monolog\Logger
99.  **monolog.logger.router** Symfony\Bridge\Monolog\Logger
100.  **monolog.logger.security** Symfony\Bridge\Monolog\Logger
101.  **monolog.logger.templating** Symfony\Bridge\Monolog\Logger
102.  **monolog.logger.translation** Symfony\Bridge\Monolog\Logger
103.  **profiler** Symfony\Component\HttpKernel\Profiler\Profiler
104.  **profiler_listener** Symfony\Component\HttpKernel\EventListener\ProfilerListener
105.  **property_accessor** Symfony\Component\PropertyAccess\PropertyAccessor
106.  **request_stack** Symfony\Component\HttpFoundation\RequestStack
107.  **response_listener** Symfony\Component\HttpKernel\EventListener\ResponseListener
108.  **router** Symfony\Bundle\FrameworkBundle\Routing\Router
109.  **router_listener** Symfony\Component\HttpKernel\EventListener\RouterListener
110.  **routing.loader** Symfony\Bundle\FrameworkBundle\Routing\DelegatingLoader
111.  **security.authentication.guard_handler** Symfony\Component\Security\Guard\GuardAuthenticatorHandler
112.  **security.authentication_utils** Symfony\Component\Security\Http\Authentication\AuthenticationUtils
113.  **security.authorization_checker** Symfony\Component\Security\Core\Authorization\AuthorizationChecker
114.  **security.csrf.token_manager** Symfony\Component\Security\Csrf\CsrfTokenManager
115.  **security.encoder_factory** Symfony\Component\Security\Core\Encoder\EncoderFactory
116.  **security.firewall** Symfony\Component\Security\Http\Firewall
117.  **security.firewall.map.context.dev** Symfony\Bundle\SecurityBundle\Security\FirewallContext
118.  **security.firewall.map.context.main** Symfony\Bundle\SecurityBundle\Security\FirewallContext
119.  **security.password_encoder** Symfony\Component\Security\Core\Encoder\UserPasswordEncoder
120.  **security.rememberme.response_listener** Symfony\Component\Security\Http\RememberMe\ResponseListener
121.  **security.token_storage** Symfony\Component\Security\Core\Authentication\Token\Storage\TokenStorage
122.  **security.user_checker.main** Symfony\Component\Security\Core\User\UserChecker
123.  **security.validator.user_password** Symfony\Component\Security\Core\Validator\Constraints\UserPasswordValidator
124.  **sensio_distribution.security_checker** SensioLabs\Security\SecurityChecker
125.  **sensio_distribution.security_checker.command** SensioLabs\Security\Command\SecurityCheckerCommand
126.  **sensio_framework_extra.cache.listener** Sensio\Bundle\FrameworkExtraBundle\EventListener\HttpCacheListener
127.  **sensio_framework_extra.controller.listener** Sensio\Bundle\FrameworkExtraBundle\EventListener\ControllerListener
128.  **sensio_framework_extra.converter.datetime** Sensio\Bundle\FrameworkExtraBundle\Request\ParamConverter\DateTimeParamConverter
129.  **sensio_framework_extra.converter.doctrine.orm** Sensio\Bundle\FrameworkExtraBundle\Request\ParamConverter\DoctrineParamConverter
130.  **sensio_framework_extra.converter.listener** Sensio\Bundle\FrameworkExtraBundle\EventListener\ParamConverterListener
131.  **sensio_framework_extra.converter.manager** Sensio\Bundle\FrameworkExtraBundle\Request\ParamConverter\ParamConverterManager
132.  **sensio_framework_extra.security.listener** Sensio\Bundle\FrameworkExtraBundle\EventListener\SecurityListener
133.  **sensio_framework_extra.view.guesser** Sensio\Bundle\FrameworkExtraBundle\Templating\TemplateGuesser
134.  **sensio_framework_extra.view.listener** Sensio\Bundle\FrameworkExtraBundle\EventListener\TemplateListener
135.  **service_container**
136.  **session** Symfony\Component\HttpFoundation\Session\Session
137.  **session.save_listener** Symfony\Component\HttpKernel\EventListener\SaveSessionListener
138.  **session.storage** alias for "session.storage.native"
139.  **session.storage.filesystem** Symfony\Component\HttpFoundation\Session\Storage\MockFileSessionStorage
140.  **session.storage.native** Symfony\Component\HttpFoundation\Session\Storage\NativeSessionStorage
141.  **session.storage.php_bridge** Symfony\Component\HttpFoundation\Session\Storage\PhpBridgeSessionStorage
142.  **session_listener** Symfony\Bundle\FrameworkBundle\EventListener\SessionListener
143.  **streamed_response_listener** Symfony\Component\HttpKernel\EventListener\StreamedResponseListener
144.  **swiftmailer.email_sender.listener** Symfony\Bundle\SwiftmailerBundle\EventListener\EmailSenderListener
145.  **swiftmailer.mailer** alias for "swiftmailer.mailer.default"
146.  **swiftmailer.mailer.default** Swift_Mailer
147.  **swiftmailer.mailer.default.plugin.messagelogger** Swift_Plugins_MessageLogger
148.  **swiftmailer.mailer.default.spool** Swift_MemorySpool
149.  **swiftmailer.mailer.default.transport** Swift_Transport_SpoolTransport
150.  **swiftmailer.mailer.default.transport.real** Swift_Transport_EsmtpTransport
151.  **swiftmailer.plugin.messagelogger** alias for "swiftmailer.mailer.default.plugin.messagelogger"
152.  **swiftmailer.spool** alias for "swiftmailer.mailer.default.spool"
153.  **swiftmailer.transport** alias for "swiftmailer.mailer.default.transport"
154.  **swiftmailer.transport.real** alias for "swiftmailer.mailer.default.transport.real"
155.  **templating** Symfony\Bundle\TwigBundle\TwigEngine
156.  **templating.filename_parser** Symfony\Bundle\FrameworkBundle\Templating\TemplateFilenameParser
157.  **templating.helper.logout_url** Symfony\Bundle\SecurityBundle\Templating\Helper\LogoutUrlHelper
158.  **templating.helper.security** Symfony\Bundle\SecurityBundle\Templating\Helper\SecurityHelper
159.  **templating.loader** Symfony\Bundle\FrameworkBundle\Templating\Loader\FilesystemLoader
160.  **templating.name_parser** Symfony\Bundle\FrameworkBundle\Templating\TemplateNameParser
161.  **translation.dumper.csv** Symfony\Component\Translation\Dumper\CsvFileDumper
162.  **translation.dumper.ini** Symfony\Component\Translation\Dumper\IniFileDumper
163.  **translation.dumper.json** Symfony\Component\Translation\Dumper\JsonFileDumper
164.  **translation.dumper.mo** Symfony\Component\Translation\Dumper\MoFileDumper
165.  **translation.dumper.php** Symfony\Component\Translation\Dumper\PhpFileDumper
166.  **translation.dumper.po** Symfony\Component\Translation\Dumper\PoFileDumper
167.  **translation.dumper.qt** Symfony\Component\Translation\Dumper\QtFileDumper
168.  **translation.dumper.res** Symfony\Component\Translation\Dumper\IcuResFileDumper
169.  **translation.dumper.xliff** Symfony\Component\Translation\Dumper\XliffFileDumper
170.  **translation.dumper.yml** Symfony\Component\Translation\Dumper\YamlFileDumper
171.  **translation.extractor** Symfony\Component\Translation\Extractor\ChainExtractor
172.  **translation.extractor.php** Symfony\Bundle\FrameworkBundle\Translation\PhpExtractor
173.  **translation.loader** Symfony\Bundle\FrameworkBundle\Translation\TranslationLoader
174.  **translation.loader.csv** Symfony\Component\Translation\Loader\CsvFileLoader
175.  **translation.loader.dat** Symfony\Component\Translation\Loader\IcuDatFileLoader
176.  **translation.loader.ini** Symfony\Component\Translation\Loader\IniFileLoader
177.  **translation.loader.json** Symfony\Component\Translation\Loader\JsonFileLoader
178.  **translation.loader.mo** Symfony\Component\Translation\Loader\MoFileLoader
179.  **translation.loader.php** Symfony\Component\Translation\Loader\PhpFileLoader
180.  **translation.loader.po** Symfony\Component\Translation\Loader\PoFileLoader
181.  **translation.loader.qt** Symfony\Component\Translation\Loader\QtFileLoader
182.  **translation.loader.res** Symfony\Component\Translation\Loader\IcuResFileLoader
183.  **translation.loader.xliff** Symfony\Component\Translation\Loader\XliffFileLoader
184.  **translation.loader.yml** Symfony\Component\Translation\Loader\YamlFileLoader
185.  **translation.writer** Symfony\Component\Translation\Writer\TranslationWriter
186.  **translator** Symfony\Component\Translation\IdentityTranslator
187.  **translator.default** Symfony\Bundle\FrameworkBundle\Translation\Translator
188.  **translator_listener** Symfony\Component\HttpKernel\EventListener\TranslatorListener
189.  **twig** Twig_Environment
190.  **twig.controller.exception** Symfony\Bundle\TwigBundle\Controller\ExceptionController
191.  **twig.controller.preview_error** Symfony\Bundle\TwigBundle\Controller\PreviewErrorController
192.  **twig.exception_listener** Symfony\Component\HttpKernel\EventListener\ExceptionListener
193.  **twig.loader** Symfony\Bundle\TwigBundle\Loader\FilesystemLoader
194.  **twig.profile** Twig_Profiler_Profile
195.  **twig.translation.extractor** Symfony\Bridge\Twig\Translation\TwigExtractor
196.  **uri_signer** Symfony\Component\HttpKernel\UriSigner
197.  **validator** Symfony\Component\Validator\Validator\ValidatorInterface
198.  **validator.builder** Symfony\Component\Validator\ValidatorBuilderInterface
199.  **validator.email** Symfony\Component\Validator\Constraints\EmailValidator
200.  **validator.expression** Symfony\Component\Validator\Constraints\ExpressionValidator
201.  **var_dumper.cli_dumper** Symfony\Component\VarDumper\Dumper\CliDumper
202.  **var_dumper.cloner** Symfony\Component\VarDumper\Cloner\VarCloner
203.  **web_profiler.controller.exception** Symfony\Bundle\WebProfilerBundle\Controller\ExceptionController
204.  **web_profiler.controller.profiler** Symfony\Bundle\WebProfilerBundle\Controller\ProfilerController
205.  **web_profiler.controller.router** Symfony\Bundle\WebProfilerBundle\Controller\RouterController
206.  **web_profiler.debug_toolbar** Symfony\Bundle\WebProfilerBundle\EventListener\WebDebugToolbarListener