Pequeño sistema para realizar registros contables (cuentas, asientos, libro diario, mayor, balance gral...)
INSTRUCTIVO: CÓDIGOS PHP EN ESTRUCTURA HTML - GUÍA PARA COMPRENDER MVC
=====================================================================

Este documento explica cómo y por qué se integran códigos PHP dentro de la estructura HTML
en las vistas del sistema contable, siguiendo el patrón de diseño MVC (Model-View-Controller).

FUNCIONALIDADES IMPLEMENTADAS:
====================================

## Sistema de Autenticación y Roles
- **Autenticación de usuarios**: Login/logout con credenciales seguras
- **Control de acceso basado en roles**: Administrador vs Usuario regular
- **Sesiones seguras**: Manejo de estado de usuario con PHP sessions
- **Protección de rutas**: Verificación de autenticación en todos los módulos

## Registro de Auditoría
- **Auditoría completa**: Todas las operaciones CRUD registradas
- **Trazabilidad**: Usuario, acción, tabla, registro y timestamp
- **Cumplimiento normativo**: Historial detallado para revisiones
- **Transparencia**: Seguimiento de cambios en el sistema

## Filtros por Fecha en Balance
- **Balance histórico**: Consulta de estados financieros por período
- **Filtros dinámicos**: Desde/hasta fechas personalizables
- **Análisis temporal**: Comparación de saldos entre períodos
- **Flexibilidad**: Vista completa o filtrada según necesidades

1. RECUERDO DEL PATRÓN MVC
==========================

MVC divide la aplicación en tres capas principales:
- **Model**: Maneja datos y lógica de negocio
- **View**: Presenta la información al usuario
- **Controller**: Gestiona las solicitudes y coordina Model y View
- **Router/Enrutador**: Punto de entrada único que dirige las solicitudes

En nuestro sistema:
- Modelos están en models/ (Cuenta.php, Asiento.php, etc.)
- Controladores en controllers/ (AsientoController.php, etc.)
- Vistas en views/ (diario.php, balance.php, etc.)
- Enrutador: index.php (punto de entrada único)

2. EL ENRUTADOR - PUNTO DE ENTRADA ÚNICO
========================================

### ¿Qué es el Enrutador?
El enrutador es el archivo principal (index.php) que recibe todas las solicitudes HTTP
y decide qué controlador debe manejar cada petición.

### ¿Por qué usar un Enrutador?
- **Punto de entrada único**: Todas las URLs pasan por index.php
- **Centralización**: Lógica de navegación en un solo lugar
- **Seguridad**: Control de acceso y validación centralizada
- **Mantenimiento**: Cambios en navegación requieren modificar solo un archivo
- **URLs limpias**: Sin exponer estructura de archivos al usuario

### Cómo funciona en nuestro sistema:

#### Estructura básica del enrutador (index.php):
```php
<?php
// Punto de entrada único
$view = $_GET['view'] ?? 'home'; // Parámetro de vista

// Enrutamiento basado en switch
switch ($view) {
    case 'cuentas':
        include __DIR__ . '/controllers/CuentaController.php';
        break;
    case 'diario':
        include __DIR__ . '/controllers/AsientoController.php';
        break;
    case 'mayor':
        include __DIR__ . '/controllers/MayorController.php';
        break;
    case 'balance':
        include __DIR__ . '/controllers/BalanceController.php';
        break;
    default:
        include __DIR__ . '/views/home.php';
        break;
}
```

#### URLs del sistema:
- `index.php?view=home` → Página principal
- `index.php?view=diario` → Libro diario
- `index.php?view=mayor&cuenta_id=1` → Mayor de cuenta específica
- `index.php?view=balance` → Balance general

### Ventajas del Enrutamiento Frontal:

#### 1. Control Centralizado:
- Todas las peticiones pasan por un solo punto
- Fácil agregar middleware (autenticación, logging, etc.)
- Validación de entrada común

#### 2. Seguridad Mejorada:
- No se exponen archivos PHP directamente
- Control de qué vistas están permitidas
- Prevención de acceso directo a controladores

#### 3. Flexibilidad:
- Cambiar estructura de URLs sin afectar controladores
- Agregar parámetros adicionales fácilmente
- Soporte para diferentes formatos de respuesta (HTML, JSON, etc.)

#### 4. Mantenimiento:
- Un solo archivo para gestionar navegación
- Fácil debugging de problemas de routing
- Cambios en flujo de aplicación centralizados

### Comparación con otros enfoques:

#### Sin Enrutador (Enfoque Antiguo):
```
proyecto/
├── cuentas.php      // Acceso directo
├── diario.php       // Acceso directo
├── mayor.php        // Acceso directo
└── balance.php      // Acceso directo
```
**Problemas:**
- Archivos expuestos directamente
- URLs feas: `/diario.php?cuenta=1`
- Difícil implementar autenticación global
- Código duplicado en cada archivo

#### Con Enrutador (Enfoque Moderno):
```
proyecto/
├── index.php        // Punto de entrada único
├── controllers/     // Lógica separada
├── models/         // Datos separados
└── views/          // Presentación separada
```
**Ventajas:**
- URLs limpias: `/?view=diario&cuenta=1`
- Controladores protegidos del acceso directo
- Separación clara de responsabilidades
- Fácil agregar funcionalidades globales

### Implementación en el Sistema:

#### Flujo de una petición típica:
1. Usuario accede a `/?view=diario`
2. index.php recibe la petición
3. Switch identifica `view=diario`
4. Incluye `AsientoController.php`
5. Controller obtiene datos de modelos
6. Controller incluye vista `diario.php`
7. Vista renderiza HTML con datos

#### Manejo de parámetros adicionales:
```php
// index.php
$view = $_GET['view'] ?? 'home';
$cuenta_id = $_GET['cuenta_id'] ?? null;
$export = $_GET['export'] ?? null;

// Pasa parámetros a controladores
switch ($view) {
    case 'mayor':
        $cuenta_id = $cuenta_id; // Disponible en el scope incluido
        include __DIR__ . '/controllers/MayorController.php';
        break;
}
```

### Mejores Prácticas para Enrutadores:

#### 1. Validación de Input:
```php
$allowed_views = ['home', 'cuentas', 'diario', 'mayor', 'balance'];
$view = $_GET['view'] ?? 'home';

if (!in_array($view, $allowed_views)) {
    $view = 'home'; // Fallback seguro
}
```

#### 2. Sanitización:
```php
$cuenta_id = filter_input(INPUT_GET, 'cuenta_id', FILTER_VALIDATE_INT);
if ($cuenta_id === false) {
    $cuenta_id = null;
}
```

#### 3. Logging:
```php
error_log("Acceso a vista: $view desde IP: " . $_SERVER['REMOTE_ADDR']);
```

#### 4. Manejo de Errores:
```php
try {
    include __DIR__ . "/controllers/{$view}Controller.php";
} catch (Exception $e) {
    error_log("Error en routing: " . $e->getMessage());
    include __DIR__ . '/views/error.php';
}
```

### Conclusión sobre el Enrutador:

El enrutador es fundamental en MVC porque:
- Proporciona un punto de entrada único y controlado
- Separa la lógica de navegación de la lógica de negocio
- Mejora la seguridad al no exponer archivos internos
- Facilita el mantenimiento y escalabilidad del sistema
- Permite URLs limpias y amigables

Sin el enrutador, tendríamos un sistema caótico con archivos expuestos
y lógica de navegación duplicada en cada controlador.

2. ¿POR QUÉ PHP EMBEBIDO EN HTML?
=================================

Las vistas usan PHP directamente embebido en HTML porque:
- **Simplicidad**: No requiere motores de template complejos
- **Rendimiento**: Menor overhead de procesamiento
- **Sintaxis familiar**: Los desarrolladores conocen PHP y HTML
- **Flexibilidad**: Permite lógica condicional y bucles directamente
- **Proyecto pequeño**: Suficiente para las necesidades actuales

3. SINTAXIS BÁSICA DE PHP EN HTML
=================================

### Etiquetas PHP:

#### Etiqueta completa: <?php ... ?>
- Para código PHP general (variables, bucles, condicionales, etc.)
- No produce output automático
- Ejemplo: `<?php $total = $a + $b; ?>`

#### Etiqueta corta de echo: <?= ... ?>
- Equivalente a `<?php echo ...; ?>`
- Diseñada específicamente para imprimir valores en HTML
- Más concisa para output simple
- Ejemplo: `<?= $variable ?>`

### ¿Cuándo usar cada una?

#### Usa <?= ... ?> cuando:
- Solo necesitas imprimir una variable o expresión simple
- Estás dentro de HTML y quieres insertar un valor dinámico
- El código es principalmente output, no lógica

#### Usa <?php ... ?> cuando:
- Necesitas lógica compleja (if, foreach, while, etc.)
- Declaras variables o realizas cálculos
- El bloque no produce output directo
- Incluyes múltiples statements

### Ejemplos en nuestro sistema:

#### En views/diario.php:
```html
<select name="cuenta_id[]">
  <?php foreach ($cuentas as $cuenta): ?>
    <option value="<?= $cuenta['id'] ?>"><?= $cuenta['nombre'] ?></option>
  <?php endforeach; ?>
</select>
```

Aquí:
- `<?php foreach (...) ?>` itera sobre el array $cuentas
- `<?= $cuenta['id'] ?>` imprime el ID de cada cuenta
- El HTML se mezcla con PHP para generar opciones dinámicas

#### En views/balance.php:
```html
<?php foreach ($cuentas as $cuenta): ?>
  <?php $saldo = Cuenta::getSaldo($cuenta['id']); ?>
  <tr data-tipo="<?= $cuenta['tipo'] ?>">
    <td><?= $cuenta['nombre'] ?></td>
    <td><?= ucfirst($cuenta['tipo']) ?></td>
    <td>$<?= number_format($saldo, 2) ?></td>
  </tr>
<?php endforeach; ?>
```

Aquí:
- Se calcula el saldo usando un método del modelo
- Se genera una fila de tabla por cada cuenta
- Los datos se formatean antes de mostrar

4. SEPARACIÓN DE RESPONSABILIDADES EN MVC
=========================================

### ¿Qué hace el Controller?
Prepara los datos y los pasa a la vista:
```php
// En AsientoController.php
$cuentas = Cuenta::all();  // Obtiene datos del modelo
$asientos = Asiento::allWithMovimientos();  // Más datos
include __DIR__ . '/../views/diario.php';  // Pasa control a la vista
```

### ¿Qué hace la View?
Solo presenta los datos preparados:
- Recibe variables como $cuentas, $asientos
- Usa bucles y condicionales para mostrar datos
- NO hace consultas a BD ni lógica de negocio
- Mantiene lógica de presentación mínima

### ¿Qué NO debe hacer la View?
- Consultas directas a base de datos
- Cálculos complejos de negocio
- Validaciones de entrada
- Lógica de aplicación

5. VENTAJAS DE ESTA ESTRUCTURA
==============================

### Para Desarrolladores:
- **Transición suave**: De HTML a PHP dinámico
- **Debugging fácil**: Errores se muestran en contexto
- **Rapidez de desarrollo**: Sin aprender nuevos motores

### Para el Sistema:
- **Mantenibilidad**: Código organizado por responsabilidades
- **Reutilización**: Vistas pueden usar los mismos datos de diferentes controllers
- **Testing**: Lógica separada facilita pruebas unitarias

### Para Rendimiento:
- **Compilación**: PHP compila el código embebido eficientemente
- **Caché**: Servidores web cachean el resultado final
- **Optimización**: Menos capas intermedias

6. EJEMPLOS PRÁCTICOS EN EL SISTEMA
===================================

### Condicionales:
```php
<?php if ($cuenta): ?>
  <h3>Movimientos de la cuenta: <?= $cuenta['nombre'] ?></h3>
  <!-- Tabla de movimientos -->
<?php else: ?>
  <p>Selecciona una cuenta</p>
<?php endif; ?>
```

### Bucles anidados (diario.php):
```php
<?php foreach ($asientos as $asiento): ?>
  <?php $movimientos = Movimiento::getByAsiento($asiento['id']); ?>
  <?php foreach ($movimientos as $index => $mov): ?>
    <tr>
      <?php if ($index === 0): ?>
        <td rowspan="<?= count($movimientos) ?>"><?= $asiento['folio'] ?></td>
      <?php endif; ?>
      <td><?= $mov['nombre'] ?></td>
      <!-- Más celdas -->
    </tr>
  <?php endforeach; ?>
<?php endforeach; ?>
```

En este ejemplo específico:
- `<?php foreach (...) ?>`: Lógica de bucle (no produce output)
- `<?php if ($index === 0): ?>`: Condicional para mostrar folio solo en primera fila
- `<td rowspan="<?= count($movimientos) ?>">`: Atributo HTML dinámico usando `<?=`
- `<?= $asiento['folio'] ?>`: Output del folio
- `<?= $mov['nombre'] ?>`: Output del nombre de cuenta

### JavaScript integrado con PHP:
```html
<script>
function exportToPDF() {
  window.open('?view=mayor&cuenta_id=<?= $cuenta_id ?>&export=pdf', '_blank');
}
</script>
```

7. MEJORES PRÁCTICAS
====================

### Organización:
- Mantener lógica PHP mínima en vistas
- Usar funciones helper para formateo
- Comentar código complejo

### Seguridad:
- Escapar output: `htmlspecialchars()` cuando sea necesario
- Validar datos en controllers, no en vistas

### Legibilidad:
- Indentar correctamente el código mezclado
- Separar bloques PHP grandes en funciones
- Usar nombres descriptivos de variables

### Ejemplo de buena práctica:
```php
// En lugar de lógica compleja en vista:
<?php
$saldo_formateado = '$' . number_format(Cuenta::getSaldo($cuenta['id']), 2);
$tipo_capitalizado = ucfirst($cuenta['tipo']);
?>

<td><?= $saldo_formateado ?></td>
<td><?= $tipo_capitalizado ?></td>
```

8. ALTERNATIVAS Y EVOLUCIÓN
===========================

### Motores de Template:
- Twig, Blade (Laravel), Smarty
- Más seguridad y separación
- Curva de aprendizaje
- Overhead adicional

### Componentes Modernos:
- Vue.js, React para frontend
- API REST con backend PHP
- Mejor UX pero más complejo

### Nuestra Elección:
- PHP embebido es suficiente para este proyecto
- Mantiene simplicidad y rendimiento
- Fácil de entender para estudiantes

10. RUTAS RELATIVAS E INCLUSIÓN DE ARCHIVOS
===========================================

### Rutas Relativas
Las rutas relativas indican la ubicación de un archivo respecto a la posición del archivo actual,
sin especificar la ruta completa del sistema de archivos.

#### Sintaxis básica:
- `../`: Sube un nivel en el directorio
- `./`: Directorio actual (opcional)
- `archivo.php`: Archivo en el mismo directorio
- `subdirectorio/archivo.php`: Archivo en subdirectorio

#### Ejemplos en nuestro sistema:
```php
// Desde controllers/AsientoController.php incluir vista
include __DIR__ . '/../views/diario.php';

// Desde views/diario.php acceder a modelo
require_once __DIR__ . '/../models/movimiento.php';
```

#### ¿Por qué usar rutas relativas?
- **Portabilidad**: El código funciona en diferentes servidores sin cambios
- **Flexibilidad**: Fácil mover directorios sin romper enlaces
- **Seguridad**: No expone estructura completa del servidor
- **Mantenimiento**: Cambios en estructura requieren menos modificaciones

### Funciones de Inclusión

#### include
- Incluye y ejecuta el archivo especificado
- Si el archivo no existe: genera WARNING y continúa ejecución
- Útil para archivos opcionales o plantillas

#### include_once
- Igual que include, pero incluye el archivo solo una vez
- Previene errores de redeclaración de funciones/clases
- Ideal para archivos que pueden ser incluidos múltiples veces

#### require
- Incluye y ejecuta el archivo especificado
- Si el archivo no existe: genera ERROR FATAL y detiene ejecución
- Útil para archivos críticos que el sistema necesita

#### require_once
- Igual que require, pero incluye el archivo solo una vez
- Más seguro para archivos críticos que pueden ser requeridos múltiples veces

### ¿Cuándo usar cada una?

#### Usa include/include_once cuando:
- El archivo es opcional (como configuraciones adicionales)
- Errores no críticos no deben detener el sistema
- Plantillas o componentes que pueden no existir

#### Usa require/require_once cuando:
- El archivo es esencial para el funcionamiento (modelos, configuraciones críticas)
- Sin el archivo, el sistema no puede continuar
- Clases o funciones que deben estar disponibles

### Ejemplos en el sistema:

#### En controladores (require_once para modelos críticos):
```php
// controllers/AsientoController.php
require_once __DIR__ . '/../models/asiento.php';     // Crítico
require_once __DIR__ . '/../models/movimiento.php'; // Crítico
require_once __DIR__ . '/../models/cuenta.php';     // Crítico
```

#### En vistas (include para mostrar contenido):
```php
// index.php (enrutador principal)
if ($view == 'diario') {
    include __DIR__ . '/controllers/AsientoController.php';
} elseif ($view == 'balance') {
    include __DIR__ . '/controllers/BalanceController.php';
}
```

#### En modelos (require_once para evitar conflictos):
```php
// models/asiento.php
require_once __DIR__ . '/../config.php'; // Configuración crítica
```

### Mejores Prácticas

#### Rutas:
- Usa `__DIR__` para rutas absolutas relativas al archivo actual
- Prefiere rutas relativas sobre absolutas
- Evita rutas que suban muchos niveles (../../../)

#### Inclusión:
- Usa `require_once` para archivos críticos (modelos, config)
- Usa `include_once` para vistas y componentes opcionales
- Organiza includes al inicio de archivos
- Documenta dependencias críticas

#### Estructura recomendada:
```
proyecto/
├── index.php (enrutador)
├── config.php (configuración global)
├── controllers/
│   └── AsientoController.php (require_once config.php y modelos)
├── models/
│   └── asiento.php (require_once config.php)
└── views/
    └── diario.php (sin includes, solo presentación)
```

11. SESIONES EN PHP - MANEJO DE ESTADO DEL USUARIO
=================================================

### ¿Qué son las Sesiones en PHP?

Las sesiones son un mecanismo que permite mantener información del usuario entre diferentes páginas web.
A diferencia de las cookies que se almacenan en el navegador del cliente, las sesiones se almacenan en el servidor,
proporcionando mayor seguridad y control sobre los datos sensibles.

### ¿Cuándo usar Sesiones?

#### Usa sesiones cuando necesites:
- **Autenticación de usuarios**: Mantener el estado de login/logout
- **Datos temporales del usuario**: Preferencias, configuración personal
- **Carritos de compra**: Mantener items seleccionados entre páginas
- **Control de acceso**: Verificar permisos y roles del usuario
- **Datos sensibles**: Información que no debe estar en el navegador del cliente

#### NO uses sesiones para:
- **Datos permanentes**: Usa base de datos para información persistente
- **Datos públicos**: Información que puede ser compartida entre usuarios
- **Archivos grandes**: Las sesiones consumen memoria del servidor

### ¿Por qué usar Sesiones?

#### Ventajas:
- **Seguridad**: Los datos se almacenan en el servidor, no en el cliente
- **Control**: El servidor puede invalidar sesiones cuando sea necesario
- **Persistencia**: Los datos sobreviven a cierres del navegador (con límites de tiempo)
- **Concurrencia**: Múltiples usuarios pueden tener sesiones simultáneas
- **Integración**: Fácil integración con bases de datos y otros sistemas

#### Comparación con Cookies:
```
COOKIES                          SESIONES
- Almacenadas en navegador        - Almacenadas en servidor
- Visibles para el usuario        - Ocultas al usuario
- Pueden ser manipuladas          - Controladas por el servidor
- Limitadas a 4KB                 - Sin límite práctico
- Persisten hasta expiración      - Persisten hasta cierre/logout
```

### Cómo usar Sesiones en PHP

#### 1. Iniciar una Sesión
Siempre debe hacerse al inicio de cada script que use sesiones:

```php
<?php
// INICIAR SESIÓN - SIEMPRE AL PRINCIPIO
session_start();

// Ahora puedes usar $_SESSION
?>
```

#### 2. Almacenar Datos en Sesión
```php
<?php
session_start();

// Almacenar datos del usuario
$_SESSION['user_id'] = 123;
$_SESSION['username'] = 'admin';
$_SESSION['role'] = 'administrator';
$_SESSION['login_time'] = time();

// Almacenar arrays complejos
$_SESSION['permissions'] = ['read', 'write', 'delete'];
$_SESSION['user_data'] = [
    'name' => 'Juan Pérez',
    'email' => 'juan@example.com',
    'last_login' => date('Y-m-d H:i:s')
];
?>
```

#### 3. Leer Datos de Sesión
```php
<?php
session_start();

// Verificar si existe la sesión
if (isset($_SESSION['username'])) {
    echo "Bienvenido, " . $_SESSION['username'];
    echo "Tu rol es: " . $_SESSION['role'];
}

// Acceder a arrays
if (isset($_SESSION['permissions'])) {
    if (in_array('write', $_SESSION['permissions'])) {
        echo "Tienes permisos de escritura";
    }
}
?>
```

#### 4. Modificar Datos de Sesión
```php
<?php
session_start();

// Actualizar datos
$_SESSION['last_activity'] = time();

// Agregar elementos a arrays
$_SESSION['permissions'][] = 'admin';

// Modificar datos existentes
$_SESSION['user_data']['last_login'] = date('Y-m-d H:i:s');
?>
```

#### 5. Eliminar Datos de Sesión
```php
<?php
session_start();

// Eliminar una variable específica
unset($_SESSION['temp_data']);

// Eliminar múltiples variables
unset($_SESSION['user_id'], $_SESSION['username']);

// Limpiar toda la sesión (manteniendo la sesión activa)
$_SESSION = [];

// Destruir completamente la sesión (logout)
session_destroy();
?>
```

### Implementación en Nuestro Sistema Contable

#### En index.php (Inicio de Sesión Global):
```php
<?php
// INICIAR SESIÓN EN TODOS LOS ARCHIVOS
session_start();

// Verificar autenticación para rutas protegidas
$view = $_GET['view'] ?? 'login';

if (!isset($_SESSION['user_id']) && $view !== 'login') {
    $view = 'login';
}

// Configurar zona horaria para sesiones
date_default_timezone_set('America/Argentina/Buenos_Aires');
?>
```

#### En login_process.php (Autenticación):
```php
<?php
session_start();

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $username = $_POST['username'] ?? '';
    $password = $_POST['password'] ?? '';

    // Validar credenciales (simplificado)
    if ($username === 'admin' && $password === 'admin123') {
        // ALMACENAR DATOS EN SESIÓN
        $_SESSION['user_id'] = 1;
        $_SESSION['username'] = 'admin';
        $_SESSION['role'] = 'admin';
        $_SESSION['login_time'] = time();
        $_SESSION['permissions'] = ['read', 'write', 'delete', 'admin'];

        header('Location: index.php?view=home');
        exit;
    }
}
?>
```

#### En Controladores (Verificación de Permisos):
```php
<?php
// controllers/CuentaController.php
session_start();

// Verificar autenticación
if (!isset($_SESSION['user_id'])) {
    header('Location: index.php?view=login');
    exit;
}

// Verificar permisos para operaciones administrativas
if ($_SESSION['role'] !== 'admin') {
    die('Acceso denegado: Se requieren permisos de administrador');
}

// Solo administradores pueden crear/editar cuentas
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    // Lógica de creación/edición
    Cuenta::create($_POST);

    // Registrar en auditoría
    Audit::log($_SESSION['user_id'], 'CREATE', 'cuentas', $new_id);
}
?>
```

#### En Vistas (Mostrar Información del Usuario):
```php
<!-- views/home.php -->
<?php if (isset($_SESSION['username'])): ?>
    <div class="user-info">
        Usuario: <?= htmlspecialchars($_SESSION['username']) ?>
        (<?= htmlspecialchars($_SESSION['role']) ?>)
        <a href="index.php?view=logout">Cerrar Sesión</a>
    </div>
<?php endif; ?>
```

#### En logout.php (Cerrar Sesión):
```php
<?php
session_start();

// Limpiar todos los datos de sesión
$_SESSION = [];

// Destruir la sesión completamente
session_destroy();

// Redirigir al login
header('Location: index.php?view=login');
exit;
?>
```

### Configuración de Sesiones

#### Archivo de Configuración (config.php):
```php
<?php
// CONFIGURACIÓN DE SESIONES
ini_set('session.cookie_httponly', 1);        // Prevenir XSS
ini_set('session.cookie_secure', 1);          // Solo HTTPS en producción
ini_set('session.use_only_cookies', 1);       // Solo cookies, no URL
ini_set('session.gc_maxlifetime', 3600);      // 1 hora de vida
ini_set('session.save_path', '/tmp/sessions'); // Directorio seguro

// Configurar nombre de sesión único para la aplicación
session_name('contabilidad_session');

// Zona horaria
date_default_timezone_set('America/Argentina/Buenos_Aires');
?>
```

#### Validación de Sesión Activa:
```php
<?php
function validarSesion() {
    session_start();

    // Verificar si existe sesión
    if (!isset($_SESSION['user_id'])) {
        return false;
    }

    // Verificar tiempo de inactividad (30 minutos)
    if (isset($_SESSION['last_activity'])) {
        $inactive_time = time() - $_SESSION['last_activity'];
        if ($inactive_time > 1800) { // 30 minutos
            session_destroy();
            return false;
        }
    }

    // Actualizar tiempo de actividad
    $_SESSION['last_activity'] = time();

    return true;
}
?>
```

### Seguridad en Sesiones

#### 1. Regenerar ID de Sesión:
```php
<?php
session_start();

// Después del login exitoso
session_regenerate_id(true); // Previene session fixation attacks
?>
```

#### 2. Validar Datos de Sesión:
```php
<?php
// Verificar integridad de datos críticos
if (!isset($_SESSION['user_id']) || !is_numeric($_SESSION['user_id'])) {
    session_destroy();
    header('Location: login.php');
    exit;
}
?>
```

#### 3. Timeout de Sesión:
```php
<?php
// Verificar tiempo de vida de la sesión
$max_session_time = 3600; // 1 hora

if (isset($_SESSION['login_time'])) {
    $session_age = time() - $_SESSION['login_time'];
    if ($session_age > $max_session_time) {
        session_destroy();
        header('Location: index.php?view=login');
        exit;
    }
}
?>
```

### Depuración de Sesiones

#### Ver Contenido de Sesión:
```php
<?php
session_start();

// Debug: Mostrar contenido de la sesión
echo "<pre>";
print_r($_SESSION);
echo "</pre>";

// Verificar si la sesión está activa
echo "Sesión activa: " . (session_status() === PHP_SESSION_ACTIVE ? 'Sí' : 'No') . "<br>";
echo "ID de sesión: " . session_id() . "<br>";
?>
```

#### Limpiar Sesiones Expiradas:
```php
<?php
// Ejecutar garbage collector manualmente
session_gc();

// Ver estadísticas
echo "Sesiones limpiadas: " . session_gc() . "<br>";
?>
```

### Mejores Prácticas

#### 1. Inicio de Sesión:
- Siempre llamar `session_start()` al principio de cada script
- Usar `session_name()` para nombres únicos por aplicación
- Configurar parámetros de seguridad en php.ini o config.php

#### 2. Almacenamiento:
- Solo almacenar datos necesarios en sesión
- No almacenar objetos complejos o datos binarios
- Usar serialización cuando sea necesario

#### 3. Seguridad:
- Regenerar ID de sesión después del login
- Usar `session_regenerate_id()` periódicamente
- Implementar timeouts de inactividad
- Validar datos de sesión antes de usarlos

#### 4. Limpieza:
- Destruir sesiones al hacer logout
- Limpiar datos sensibles al cerrar sesión
- Implementar garbage collection automático

#### 5. Rendimiento:
- Minimizar datos almacenados en sesión
- Usar bases de datos para datos persistentes
- Configurar `session.save_path` en directorio con permisos adecuados

### Conclusión sobre Sesiones

Las sesiones son fundamentales en aplicaciones web PHP porque:
- Permiten mantener estado entre peticiones HTTP (que son stateless)
- Proporcionan seguridad al almacenar datos sensibles en el servidor
- Facilitan la implementación de autenticación y control de acceso
- Son esenciales para aplicaciones interactivas modernas

En nuestro sistema contable, las sesiones nos permiten:
- Mantener usuarios autenticados entre páginas
- Controlar permisos basados en roles
- Registrar auditoría con información del usuario actual
- Proporcionar una experiencia de usuario fluida

Sin sesiones, cada página requeriría re-autenticación, haciendo imposible
la navegación fluida y el control de acceso basado en usuarios.

9. CONCLUSIÓN
=============

La integración de PHP en HTML en las vistas MVC permite:
- Desarrollo rápido y familiar
- Separación clara de responsabilidades
- Código mantenible y escalable
- Optimización de rendimiento

Siguiendo estas prácticas, el código permanece limpio y funcional,
facilitando el aprendizaje y mantenimiento del sistema contable.
