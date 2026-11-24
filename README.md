🏘️ Junta de Vecinos Zaror — App de arriendos y transparencia financiera

Aplicación móvil Android (Kotlin + Jetpack Compose) conectada a 2 microservicios Spring Boot y a una API externa (mindicador.cl). Proyecto de la asignatura DSY1105 Desarrollo de Aplicaciones Móviles (DUOC UC) — Evaluación Parcial 4.

1. Nombre del proyecto

Junta de Vecinos Zaror (com.example.androidexample) — gestión de reservas de la sede vecinal, ingresos/gastos y transparencia.

2. Integrantes
Integrante	      Rol
Matías Letelier	  Desarrollo completo: app móvil, microservicios, pruebas unitarias, firma del APK y documentación

4. Funcionalidades
Perfil	Qué puede hacer
Todos	Registro e inicio de sesión (clave con hash + salt), ver Transparencia (ingresos, gastos, saldo, buscador y filtros), ver indicadores económicos (UF, dólar, euro)
Vecino	Solicitar reserva de la sede (formulario validado), ver Mis reservas por estado, cancelar solicitudes pendientes
Secretaría	Ver todas las reservas, aprobar / rechazar (no permite dos aprobadas el mismo día) y notificación local
Tesorería	Registrar movimientos con foto de la boleta (cámara), editar y eliminar movimientos

CRUD completo contra los microservicios: Crear (reserva/movimiento) · Reer (listas) · Update (aprobar/rechazar, editar movimiento) · Delete (cancelar reserva, eliminar movimiento).

Usuarios de prueba

La app crea estas cuentas automáticamente la primera vez que se abre (clave de todas: Junta1234):

Perfil	Correo
Vecino	vecino@junta.cl
Secretaría	secretaria@junta.cl
Tesorería	tesoreria@junta.cl

También se pueden crear cuentas nuevas desde Crear cuenta.

Arquitectura
App Android (Compose + ViewModel + Retrofit)
   ├── Room (SQLite local) ........ usuarios / login
   ├── reservas-service  :8081 .... Spring Boot + JPA + H2  (api/reservas)
   ├── finanzas-service  :8082 .... Spring Boot + JPA + H2  (api/movimientos)
   └── mindicador.cl (HTTPS) ...... API externa: UF, dólar, euro
4. Endpoints utilizados
Microservicio reservas-service — http://10.0.2.2:8081
Método	Ruta	Descripción
GET	/api/reservas	Lista todas (opcional ?usuarioId=)
GET	/api/reservas/{id}	Obtiene una
GET	/api/reservas/ocupada?fecha=dd/MM/yyyy	{"ocupada": true/false}
POST	/api/reservas	Crea (queda PENDIENTE)
PUT	/api/reservas/{id}	Actualiza datos
PATCH	/api/reservas/{id}/estado?valor=APROBADA	Cambia estado (409 si la fecha ya está aprobada)
DELETE	/api/reservas/{id}	Elimina (204)
Microservicio finanzas-service — http://10.0.2.2:8082
Método	Ruta	Descripción
GET	/api/movimientos	Lista movimientos
GET	/api/movimientos/resumen	{ingresos, gastos, saldo}
GET	/api/movimientos/{id}	Obtiene uno
POST	/api/movimientos	Crea (tipo: INGRESO / GASTO)
PUT	/api/movimientos/{id}	Actualiza
DELETE	/api/movimientos/{id}	Elimina (204)
API externa — https://mindicador.cl/api
Método	Ruta	Uso en la app
GET	/api	UF, dólar y euro; se muestran en Transparencia y se calcula el saldo aproximado en dólares

10.0.2.2 es el localhost del PC visto desde el emulador. Con celular físico, cambiar HOST en app/src/main/java/com/example/androidexample/data/Remote/ApiConfig.kt por la IP del PC.

5. Pasos para ejecutar

Requisitos: JDK 17, Maven 3.9+, Android Studio (última versión), emulador o celular.

Clonar el repositorio.
Levantar los microservicios (cada uno en su terminal):
bash
cd backend/reservas-service && mvn spring-boot:run     # http://localhost:8081
cd backend/finanzas-service && mvn spring-boot:run     # http://localhost:8082

Consola de la BD (H2): http://localhost:8081/h2-console y :8082/h2-console
JDBC URL: jdbc:h2:file:./data/reservasdb (o finanzasdb) · usuario sa · sin clave. 3. Abrir la app en Android Studio (carpeta raíz del repo) → Sync Gradle → ejecutar en el emulador. 4. Iniciar sesión con las cuentas de prueba de cada perfil (ver Usuarios de prueba) y probar.

Pruebas unitarias
bash
./gradlew testDebugUnitTest

Con cobertura: en Android Studio, clic derecho en app/src/test → Run Tests with Coverage.
Herramientas: JUnit 4 + MockK + kotlinx-coroutines-test. Cubren ViewModels (Auth, Reserva, Finanzas, Indicadores), repositorios, validadores, hasher de claves y manejo de errores de red.

Generar el APK firmado
bash
./gradlew assembleRelease
# → app/build/outputs/apk/release/app-release.apk

La firma está configurada en app/build.gradle.kts (signingConfigs.release) con el keystore keystore/junta-release.jks
(alias junta). Para regenerarlo:

bash
keytool -genkeypair -v -keystore keystore/junta-release.jks -alias junta -keyalg RSA -keysize 2048 -validity 10000

Verificar la firma: apksigner verify --print-certs app/build/outputs/apk/release/app-release.apk
