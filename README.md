# entregable-1-cicd

Primer entregable del curso de CI/CD.

## Integrantes

Nombre usado en AUTOR y en el saludo del workflow: **Samuel Cadavid Zapata**

| Nombre completo | Usuario de GitHub | Rol |
|---|---|---|
| Samuel Cadavid Zapata | [scadavidzea](https://github.com/scadavidzea) | Propietario del repositorio |

Entrega individual.

## Ejecución del workflow

Ejecución exitosa de `mi_workflow.yml` con el workflow y el script entregados:
https://github.com/scadavidzea/entregable-1-cicd/actions/runs/42383525493

También lo probé de forma manual con "Run workflow":
https://github.com/scadavidzea/entregable-1-cicd/actions/runs/42383553350

En los logs sale el saludo con mi nombre, la fecha y hora del runner, y las tres líneas del script de Python. El sistema operativo que muestra es Linux, aunque yo trabajo en Windows.

## Preguntas

### 1. ¿Qué eventos configuraste para disparar tu workflow y por qué?

Configuré dos eventos:

- `push` a la rama `main`: así el workflow corre solo cada vez que subo cambios. Es la idea principal de automatizar, no tener que acordarme de correrlo.
- `workflow_dispatch`: me deja correrlo a mano desde la pestaña Actions con el botón "Run workflow". Sirve para probarlo otra vez sin tener que hacer un commit solo para eso.

Le puse `branches: main` al push para que solo se ejecute con los cambios de la rama principal y no con cualquier rama de pruebas.

### 2. ¿Qué sistema operativo usa el runner ubuntu-latest y por qué funciona aunque mi computador sea Windows? ¿Qué relación tiene con el shell?

`ubuntu-latest` es una máquina virtual con Ubuntu Linux que pone GitHub. El workflow no se ejecuta en mi computador, se ejecuta en esa máquina en la nube. Yo desde Windows solo subo el código con `git push` y GitHub se encarga de crear la máquina, correr los pasos y borrarla al final. Por eso no tuve que instalar Ubuntu ni nada, y por eso el script de Python dice "Linux" y no "Windows".

El runner define qué shell se usa por defecto en los pasos `run`. En Ubuntu es bash y en Windows es PowerShell. Por eso funciona `$(date)` para sacar la fecha, que es un comando de bash. Si cambiara el runner a `windows-latest` ese comando ya no serviría igual y tocaría usar algo como `Get-Date`. Igual en el paso del mensaje dejé `shell: bash` para que quede claro.

### 3. Tres acciones del Marketplace de GitHub Actions

- **actions/setup-node**: instala la versión de Node.js que uno le diga en el runner. Sirve para proyectos de JavaScript, por ejemplo para correr `npm install` y `npm run build` de una página web antes de publicarla.
- **actions/cache**: guarda carpetas entre una ejecución y otra, como las dependencias de npm o pip. Así no se descarga todo cada vez y el workflow termina más rápido.
- **docker/build-push-action**: construye una imagen de Docker a partir del Dockerfile del proyecto y la puede subir a un registro como Docker Hub o GitHub Container Registry. Sirve para dejar lista la aplicación para desplegarla en un servidor.

### 4. ¿Qué es un secreto en GitHub Actions y para qué se usa?

Un secreto es un valor privado, como una contraseña, un token o una llave de una API, que se guarda en la configuración del repositorio (Settings > Secrets and variables > Actions) y no en el código. Se usa porque el repositorio es público y cualquiera podría ver una clave si la escribo directamente en el YAML.

En el workflow se llama con `${{ secrets.NOMBRE_DEL_SECRETO }}`. Por ejemplo, si quisiera que me llegara un mensaje a Telegram cuando el workflow termine, guardaría el token del bot como secreto y lo pasaría al paso así:

```yaml
- name: Avisar que terminó
  env:
    TOKEN_BOT: ${{ secrets.TOKEN_BOT }}
  run: python avisar.py
```

Lo importante es no imprimirlo nunca con `echo`. GitHub intenta ocultarlo en los logs, pero eso no es una garantía. Para este taller no necesité ningún secreto.

### 5. Reflexión: ¿qué automatizaría en mi trabajo diario?

**Samuel Cadavid Zapata:**

La primera cosa que automatizaría es la publicación de la página web de Roundtrip Medellín, una agencia de tours en la que estoy trabajando el rediseño. Hoy cada cambio lo tengo que construir y subir a mano. Con un workflow que se ejecute con cada push a `main`, se construiría la página y se publicaría sola, y si el build falla no se publica nada. Así no me arriesgo a dejar la página caída por un error y me ahorro hacer ese proceso cada vez.

La segunda es mejorar la calidad de los despliegues del sistema de ventas que estoy haciendo para Apple Travel. Ese proyecto ya corre las pruebas en GitHub Actions en cada push, pero el despliegue al servidor todavía lo lanzo yo manualmente. Lo que haría es que el despliegue solo se pueda hacer si todas las pruebas pasaron, que antes de desplegar se saque una copia de la base de datos y que después se revise automáticamente que la aplicación sí quedó respondiendo. Aquí no pondría despliegue continuo sino entrega continua, con una aprobación manual antes de pasar a producción, porque el sistema maneja los pagos a los operadores y un error ahí es plata real que no se puede perder.
