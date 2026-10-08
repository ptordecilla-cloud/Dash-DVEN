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

La carga reemplaza los datos maestros APIS, alimenta el CAPEX, gráfico, tabla filtrable y detalle con las doce columnas. Presupuesto anual, gasto, avances mensuales, cronograma y riesgos quedan **sin datos**: no se deducen ni se mezclan con cifras ficticias. La integración de CD agosto queda pendiente de su estructura y unidades. APIS no proporciona cortes mensuales, por lo que esta carga no crea historial mensual financiero.

Los Excel se procesan localmente con ExcelJS. Los registros quedan en `sessionStorage`, separados de la demostración, durante la sesión de esa pestaña. El botón Eliminar datos importados borra la cartera. No hay subida de archivos a un servidor, sincronización con Drive ni datos corporativos incluidos en el código o GitHub. En equipos compartidos elimina los datos importados al finalizar.
