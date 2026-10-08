# Dashboard Ejecutivo DVEN

Aplicación web de demostración para una cartera de inversiones. **Todos los proyectos, responsables y cifras son ficticios.** No contiene credenciales ni datos corporativos. Los montos se expresan en millones de pesos chilenos (MM$ CLP).

## Desarrollo

Requiere Node.js 22 o 24 y npm. Desde la raíz del repositorio:

```sh
npm ci --cache /tmp/dven-npm-cache
npm run dev
```

```sh
npm run build
npm test
npm run preview
```

La compilación verifica TypeScript y genera `dist/`. Las pruebas validan importación, conservación del historial y reglas de semáforo.

## Vistas y métricas

Menú: Resumen Ejecutivo, Cartera de Proyectos, Presupuesto, Avance Físico, Cronograma, Riesgos y Detalle de Proyecto. Selector mensual global, filtros de área/estado/texto, gráficos con tooltip y selección de proyecto mediante tabla o selector y botones anterior/siguiente.

CAPEX es la inversión total de los proyectos con datos en el corte. Presupuesto anual corresponde a la asignación anual por proyecto; gasto real es acumulado al mes seleccionado. Avance físico ejecutivo: promedio ponderado por presupuesto anual. Crítico: real menos programado < −10 puntos porcentuales; Atención: < −5; En línea: ≥ −5. Reformulación es una bandera independiente declarada en el corte. El cronograma muestra fechas planificadas; no calcula pronósticos de término. Los gráficos de evolución muestran todo el historial disponible, incluso cortes posteriores al seleccionado.

## Arquitectura y CSV

- `src/data.ts`: modelos `Project`, `Snapshot`, `Portfolio`, datos ficticios, reglas y funciones puras de importación/exportación.
- `src/App.tsx`: navegación, filtros, métricas, gráficos y persistencia local.
- `src/style.css`: Tailwind CSS 4 y estilos adaptables.
- `src/data.test.ts`: pruebas del contrato de datos.

`Project` mantiene atributos maestros; `Snapshot` conserva un corte por `(projectId, month)`. La versión del esquema es 1. Los cortes importados se guardan en `localStorage` (`dven-demo-v1`) solamente en este navegador; no hay backend, autenticación ni sincronización. Restablecer requiere confirmación y recupera los datos iniciales. Para un backend futuro, reemplazar la carga/persistencia local por un repositorio de datos con validación del mismo contrato y control de acceso.

Exportar CSV también proporciona una plantilla. Se aceptan archivos UTF-8 separados por coma, máximo 2 MB, con cabecera exacta:

```csv
projectId,month,budget,actual,planned,physical,reformulation
DV-001,2026-07,18000,17000,70,63,false
```

Solo importar **datos ficticios de prueba**. Los IDs deben existir en los proyectos maestros. Mes `YYYY-MM`, montos finitos no negativos, avances entre 0 y 100, bandera `true`/`false`. La importación es atómica; duplicados dentro del archivo se rechazan. Un corte existente con la misma clave se reemplaza; otros meses se conservan. Nuevos meses aparecen en el selector. La importación de proyectos maestros queda como extensión futura.

## Publicar en Vercel

1. Sube este repositorio a GitHub y crea un proyecto en Vercel importándolo.
2. Selecciona framework **Vite** y raíz del proyecto donde está `package.json`.
3. Usa Node.js **24.x**, instalación `npm ci`, build `npm run build` y directorio de salida `dist`.
4. No se requieren variables de entorno ni secretos. Publica y comprueba las siete vistas y el selector mensual.

La navegación vive en estado React y no utiliza rutas de servidor: no requiere reglas de rewrite. El historial local depende del navegador y dominio; no se comparte entre usuarios ni entre dominios de preview y producción. La publicación no se realiza automáticamente desde este repositorio.

## Carga Excel APIS (datos importados)

El botón **Cargar Excel APIS / datos importados** abre un modo separado de la demostración. Selecciona un `.xlsx` (máximo 10 MB), elige la hoja, revisa la vista previa y confirma. Para archivos `.xls`, guarda una copia `.xlsx` en Excel. Se busca una fila de encabezados dentro de A–L, normalizando acentos y espacios:

A Código de Proyecto; B Nombre; C Año de Presentación; D Total Proyecto KUSD; E Tipo Decisión Codelco; F Justificación; G Etapa; H Gestor-Ejecutor; I Área; J División; K Descripcion; L Proposito.

Las columnas posteriores a L no se importan. Se validan códigos únicos, nombre, año y montos no negativos. Celdas numéricas se usan sin conversión; textos como `1.616` se interpretan como 1616 KUSD y `1.616,50` como 1616,50 KUSD. Fórmulas requieren un resultado calculado y guardado en Excel: el navegador no recalcula el libro. Una carga inválida conserva la cartera anterior.

La carga reemplaza los datos maestros APIS, alimenta el CAPEX, gráfico, tabla filtrable y detalle con las doce columnas. Hasta cargar Flash, presupuesto anual, gasto, avances mensuales, cronograma y riesgos quedan **sin datos**: no se deducen ni se mezclan con cifras ficticias. APIS no proporciona cortes mensuales, por lo que su carga no crea historial financiero. La carga Flash descrita más abajo incorpora ese control en KUSD; CD agosto queda pendiente de su estructura.

Los Excel se procesan localmente con ExcelJS. Los registros quedan en `sessionStorage`, separados de la demostración, durante la sesión de esa pestaña. El botón Eliminar datos importados borra la cartera. No hay subida de archivos a un servidor, sincronización con Drive ni datos corporativos incluidos en el código o GitHub. En equipos compartidos elimina los datos importados al finalizar.

## Control mensual Flash · KUSD

En el modo de datos importados, **Cargar Excel Flash** acepta `.xlsx` de hasta 10 MB. Lee A–AH (34 columnas), selecciona la hoja, detecta `Mes de Control` (por ejemplo `agosto-26`) y permite confirmar/corregir el mes antes de aplicar la vista previa. El formato es el encabezado Flash proporcionado: API en A, Nombre en B, Prog del total API en H, Inicio API en AE y Término máximo sin reformular en AH. No requiere ni lee información de Google Drive.

Correspondencia de columnas (base 1):

| Campo | Columna |
| --- | --- |
| Total API programado / real-proyectado | H / I |
| Acumulado total financiero / físico | K / L |
| Presupuesto anual / proyección anual financiera | M / N |
| Avance físico anual programado / real-proyectado | P / Q |
| Financiero enero–mes control programado / real | S / T |
| Físico enero–mes control programado / real | V / W |
| Gasto del mes programado / real | Y / Z |
| Físico del mes programado / real | AB / AC |
| Inicio / término autorizado / real-proyectado / máximo | AE / AF / AG / AH |

Todos los montos Flash se interpretan en KUSD, confirmado por el usuario. Se aceptan números Excel y textos chilenos con `$`, puntos de miles y coma decimal. Porcentajes numéricos Excel deben guardarse como fracciones (0,291 = 29,1%); textos pueden usar `29,1%`. Las fechas pueden ser celdas de fecha Excel, DD-MM-YYYY o YYYY-MM-DD. Las fórmulas requieren valores calculados guardados. No se ejecutan macros ni se recalculan fórmulas.

El CAPEX APIS y el total API Flash se muestran separados; no se convierten ni se reemplazan automáticamente. Presupuesto anual usa M; gasto acumulado usa T, no N ni Z. Avance físico ejecutivo usa V/W ponderado por M, solo para proyectos con ambos avances y presupuesto positivo. Se indica cobertura: vacíos son sin datos, no cero. No se utiliza Cumplimiento como porcentaje físico. Críticos: avance real menos programado < −10 puntos; atención: < −5. Alerta de reformulación por fecha: AG > AH; fechas faltantes dejan alerta sin datos. Es una señal temporal, no una evaluación formal.

Cada registro es `(API, mes)`. Recargar la misma clave reemplaza solo ese corte. Los otros meses y códigos se conservan en `sessionStorage` (`dven-flash-v1`) durante la sesión de esa pestaña, incluyendo recargas de página. Exportar historial CSV permite conservar una copia fuera del navegador; ese CSV es un respaldo de consulta, no el formato de importación Flash. La persistencia compartida/permanente requerirá un backend futuro. La selección de corte actualiza KPIs, tablas, presupuesto y cronograma. Detalle muestra la comparación APIS/Flash y el historial por proyecto. Los proyectos sin APIS aparecen en la tabla Flash y en el selector de detalle como pendientes, sin inventar año ni CAPEX APIS.

Flash y APIS se cargan de forma independiente y en cualquier orden. Los datos del usuario no están incrustados en el repositorio. CD agosto sigue pendiente de su estructura; no debe cargarse como Flash a menos que tenga exactamente ese diseño.
