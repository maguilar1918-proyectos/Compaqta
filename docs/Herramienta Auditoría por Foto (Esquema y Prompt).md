# Herramienta Auditoría por Foto — Esquema y Prompt

Servicio incluido en la consultoría de Compaqta: el equipo del cliente fotografía un hallazgo en piso y la plataforma arma el hallazgo formal, el plan CAPA, el seguimiento y los informes. El auditor de Compaqta valida antes de que algo llegue al cliente.

> Los tres documentos de referencia (reporte de inspección, resumen gerencial y presentación de reunión HACCP) pertenecen a un cliente real. Se usaron solo para entender el esquema. No se publican en la web ni se guardan en este repositorio. Todo ejemplo público debe ser ficticio y marcarse como EJEMPLO.

## 1. Flujo

| Etapa | Quién | Qué ocurre | Resultado |
|---|---|---|---|
| 1. Captura | Inspector del cliente | Foto desde el celular, área y comentario opcional | Hallazgo en borrador con foto, área, fecha e inspector |
| 2. Análisis | IA (visión) | Propone requisito infringido, evidencia objetiva, tipo de peligro y clasificación | Hallazgo estructurado con nivel de confianza |
| 3. Validación | Auditor de Compaqta | Confirma o corrige clasificación y requisito | Hallazgo validado con historial de cambios |
| 4. Plan CAPA | IA + responsable del cliente | Corrección inmediata, causa raíz y acción correctiva, con responsable y fecha límite | Hallazgo abierto con plazo |
| 5. Cierre | Responsable | Sube la foto "después" | Estado: por verificar |
| 6. Verificación | Calidad | Aprueba o rechaza el cierre | Estado: verificado o reabierto |
| 7. Informes | Plataforma | Genera los tres informes de la sección 4 | PDF, PPTX y tablero |

Estados del hallazgo: borrador, validado, abierto, por verificar, verificado, vencido, reincidente.

## 2. Clasificación y calificación

Hay dos modos y no deben mezclarse.

| Modo | Uso | Clasificación | Calificación |
|---|---|---|---|
| Inspección interna de planta (BPM, orden y limpieza, plagas) | Recorridos semanales o quincenales | Crítica, Mayor, Menor | Puntaje ponderado: peso por criticidad, reincidente x2, escala AA a Reprobado configurable por cliente |
| Auditoría contra norma (BRCGS, SQF, FSSC 22000, SMETA, Fairtrade, Orgánico) | Preparación o simulacro de auditoría de tercera parte | Según la norma: No Conformidad Crítica, Mayor o Menor en BRCGS; Área de Cumplimiento en SMETA | Según el esquema de la norma |

Criterios base de clasificación (ajustables por cliente):

| Nivel | Criterio |
|---|---|
| Crítica | Riesgo directo e inmediato a la inocuidad, a la legalidad del producto o a las personas. Detiene producción o liberación |
| Mayor | Falla significativa de un control, o ausencia de un requisito, con riesgo probable si no se corrige |
| Menor | Desviación puntual que no compromete por sí sola el control del peligro |

## 3. Prompt del sistema (modelo de visión)

Parámetros por proyecto: `{NORMA}`, `{SECTOR}`, `{AREAS}`, `{ESCALA}`, `{IDIOMA}`.

```
Eres un Auditor Líder Multiestándar de Compaqta. Analizas una foto tomada en
planta o campo y redactas un hallazgo formal bajo la norma {NORMA}, sector
{SECTOR}. Responde en {IDIOMA} con terminología oficial exacta de la norma.

ENTRADA
- foto (obligatoria), área (de la lista {AREAS}), comentario del inspector
  (opcional) y hallazgos previos del área (para detectar reincidencia).

REGLAS
1. Describe solo lo que se ve. No inventes datos que no estén en la foto o en
   el comentario. Si la foto no alcanza para concluir, devuelve
   "requiere_mas_informacion" y di qué foto o dato falta.
2. Redacta el hallazgo con tres partes: requisito infringido (cláusula o
   criterio de {NORMA}), evidencia objetiva (hecho observable, ubicación y
   magnitud) y clasificación (Crítica, Mayor o Menor según {ESCALA}).
3. Identifica el tipo de peligro: químico, físico, biológico, plagas, fraude,
   defensa alimentaria, derechos humanos, seguridad laboral u otro.
4. Justifica la clasificación en una línea. Ante duda entre dos niveles,
   propone el mayor y marca "revision_humana": true.
5. Propón el plan CAPA separando:
   - correccion_inmediata: qué hacer hoy para contener el riesgo.
   - accion_correctiva: causa raíz probable y acción que evita recurrencia.
   Sugiere responsable por cargo (no por nombre) y plazo según criticidad.
6. Si el hallazgo es de organico o comercio justo, evalúa segregación y
   balance de masas antes de cerrar la clasificación.
7. Si aparecen rostros o datos personales, indícalo en "privacidad" para
   difuminarlos antes de incluir la foto en informes.
8. Nunca declares cumplimiento ni emitas dictamen de certificación. Tu salida
   es una propuesta que valida un auditor de Compaqta.

SALIDA (solo JSON válido)
{
  "estado_analisis": "completo | requiere_mas_informacion",
  "area": "",
  "item": "",
  "requisito_infringido": { "norma": "", "clausula": "", "texto": "" },
  "evidencia_objetiva": "",
  "tipo_peligro": [""],
  "clasificacion": "Critica | Mayor | Menor",
  "justificacion_clasificacion": "",
  "reincidencia_probable": false,
  "capa": {
    "correccion_inmediata": "",
    "accion_correctiva": "",
    "responsable_sugerido": "",
    "plazo_dias": 0
  },
  "confianza": 0.0,
  "revision_humana": true,
  "privacidad": { "rostros": false, "documentos": false },
  "faltante": ""
}
```

Prompt de verificación (foto "después"):

```
Compara la foto "antes" y la foto "después" del hallazgo {CODIGO}. Indica si
la corrección elimina la evidencia objetiva y contiene el riesgo. Responde
solo JSON: { "resultado": "cumple | no_cumple | no_concluyente",
"motivo": "", "diferencias_observadas": [""], "revision_humana": true }.
Si el ángulo o la luz no permiten comparar, responde "no_concluyente".
```

## 4. Documentos que genera

| Documento | Formato | Contenido |
|---|---|---|
| Reporte de inspección | PDF | Portada con inspectores y fecha, KPI (áreas con hallazgos, hallazgos, puntos, calificación), barra por criticidad, resumen por área, ficha por hallazgo con foto antes y después, responsable, fecha límite y estado, y firmas |
| Resumen gerencial | PDF | Mensaje principal, KPI contra el período anterior, evolución por recorrido, hallazgos por criticidad y por área, cierre en plazo, días promedio de cierre, reincidentes, críticos y recomendaciones priorizadas |
| Presentación de reunión HACCP | PPTX | Calificación y escala, evolución, concentración por área, avance de cierre por responsable y por área, antes y después de lo corregido, una lámina por hallazgo abierto y tabla de acuerdos |

Código de hallazgo: `H-AAMMDD-NN`. Cada documento lleva código de formato, versión y fecha de generación.

## 5. Modelo de datos mínimo

| Entidad | Campos |
|---|---|
| Recorrido | id, fecha, inspectores, norma, calificación, puntaje |
| Hallazgo | código, recorrido, área, ítem, foto antes, evidencia, requisito, clasificación, peso, responsable, fecha límite, estado, reincidente, historial de cambios |
| Cierre | hallazgo, foto después, fecha, resultado de verificación, verificador, ampliación de plazo |
| Cliente y configuración | norma, escala, pesos, áreas, responsables, logo |

## 6. Controles de calidad y riesgos

| Riesgo | Control |
|---|---|
| La IA clasifica mal | Validación obligatoria del auditor de Compaqta; el cliente ve el estado "validado" |
| Cambio de clasificación entre informes sin explicación | Historial de cambios con motivo y quién lo hizo |
| Confidencialidad de fotos y datos del cliente | Datos separados por cliente, rostros difuminados, sin uso para entrenar modelos |
| Fotos insuficientes | Estado "requiere más información" en lugar de adivinar |
| Dependencia de un proveedor de IA | Prompt y esquema JSON independientes del modelo |

## 7. Preguntas abiertas

1. En los documentos de referencia, el mismo recorrido aparece con 14 hallazgos Mayor y 69 puntos en un documento, y con 3 Mayor y 47 puntos en otro. Hay hallazgos que pasan de Mayor a Menor entre documentos. Definir si es una reclasificación y cómo se registra.
2. Moneda y precio de referencia del servicio (la meta de 5k por mes: soles o dólares).
3. Si la plataforma se construye como producto propio (app) o se opera inicialmente de forma asistida por Compaqta.
