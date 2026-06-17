# TiddlyWiki5 Vista previa del enlace al pasar el mouse - Informe de investigacion de mejores practicas

## 1. Investigacion sobre el mecanismo central.

### 1.1 EventCatcher Widget (Metodos recomendados)

**EventCatcher Widget** es TiddlyWiki v5.1.23+ El mecanismo avanzado de procesamiento de eventos introducido puede capturar DOM Evento y desencadenante Action Widgets。

**Fortalezas centrales：**
- ✅ puro wikitext Implementacion, no es necesaria JavaScript startup module
- ✅ Excelente desempeno (mecanismo de delegacion de eventos）
- ✅ Apoyo `mouseover`、`mouseout` Espera a todos DOM Eventos
- ✅ Proporcionar variables ricas (coordenadas、DOM Propiedades, etc.）

**Gramatica basica：**
```wikitext
<$eventcatcher 
  selector=".tc-tiddlylink" 
  $mouseover=<<show-actions>> 
  $mouseout=<<hide-actions>>
  tag="div"
>
  <!-- Area de contenido -->
</$eventcatcher>
```

**Variables disponibles：**
- `dom-*`: Combina todos los elementos DOM Propiedades
- `event-type`: Tipo de evento
- `tv-popup-coords`: Coordenadas relativas (utilizadas para posicionar ventanas emergentes）
- `tv-popup-abs-coords`: Coordenadas absolutas（v5.2.4+）
- `tv-selectednode-*`: Seleccione el tamano y la posicion del nodo.
- `event-fromviewport-pos*`: Posicion del evento en relacion con la ventana grafica.

### 1.2 Reveal Widget + Popup Mecanismo

**Reveal Widget** Se utiliza para mostrar contenido de forma condicional, junto con popup Mecanismo para realizar ventanas emergentes.。

**popup Escribe Reveal：**
```wikitext
<$reveal type="popup" state="$:/state/myPopup" position="below">
  <div class="tc-popup-keep">
    Contenido de la ventana emergente
  </div>
</$reveal>
```

**Atributos importantes：**
- `type="popup"`: Habilitar popup Posicionamiento
- `position`: Ubicacion（left, above, aboveleft, aboveright, right, belowleft, belowright, below）
- `class="tc-popup-keep"`: Al hacer clic dentro no se cierra（sticky popup）
- `retain="yes"`: Conservar el contenido al ocultarlo (con animate）
- `animate="yes"`: Efectos de animacion (requeridos retain="yes"）

### 1.3 Action-Popup Widget

**Action-Popup Widget** Se utiliza para activar o cerrar ventanas emergentes.。

```wikitext
<$action-popup 
  $state="$:/state/myPopup" 
  $coords=<<tv-popup-coords>> 
  $floating="no"
/>
```

**Parametros importantes：**
- `$state`: Estado tiddler Titulo de
- `$coords`: Cadena de coordenadas (relativa o absoluta)）
- `$floating`: Si se trata de una ventana emergente flotante (debe cerrarse explicitamente)）
- Si `$coords` Si esta vacio, cierre todas las ventanas emergentes.

### 1.4 Sistema de coordenadas (v5.2.4+)

**Formato de coordenadas relativas：** `(left,top,right,bottom)`
- Cuadro delimitador relativo al elemento desencadenante

**Formato de coordenadas absolutas：** `[(left,top,width,height)]`
- Posicion absoluta relativa a la ventana grafica

---

## Segundo, puro Wikitext Plan de implementacion

### Planificar A: EventCatcher + Reveal Widget（Recomendar）

Esto es**Lo mas moderno y elegante**El metodo de implementacion es completamente innecesario. JavaScript。

#### 2.1 PageTemplate Tiddler

Crear `$:/plugins/yourname/link-preview/page-template`

```wikitext
title: $:/plugins/yourname/link-preview/page-template
tags: $:/tags/PageTemplate

\define show-preview-actions()
<$action-popup 
  $state="$:/state/link-preview/popup" 
  $coords=<<tv-popup-coords>>
/>
<$action-setfield 
  $tiddler="$:/state/link-preview/current" 
  $field="text" 
  $value=<<dom-href>>
/>
<$action-setfield 
  $tiddler="$:/state/link-preview/current" 
  $field="link-title" 
  $value=<<dom-data-tiddler-title>>
/>
\end

\define hide-preview-actions()
<$action-popup 
  $state="$:/state/link-preview/popup"
/>
\end

<$eventcatcher
  selector=".tc-tiddlylink"
  $mouseover=<<show-preview-actions>>
  $mouseout=<<hide-preview-actions>>
  tag="div"
  class="link-preview-catcher"
>
  <$reveal 
    type="popup" 
    state="$:/state/link-preview/popup"
    position="belowleft"
    class="tc-popup-keep link-preview-popup"
    animate="yes"
    retain="yes"
  >
    <div class="link-preview-container">
      <$tiddler tiddler={{$:/state/link-preview/current!!link-title}}>
        <$transclude tiddler="$:/plugins/yourname/link-preview/template" mode="block"/>
      </$tiddler>
    </div>
  </$reveal>
  
  {{||$:/core/ui/PageTemplate}}
</$eventcatcher>
```

**Conclusiones clave：**
1. ✅ uso `$:/tags/PageTemplate` Las etiquetas envuelven todo el contenido a nivel de pagina.
2. ✅ EventCatcher Captura todo `.tc-tiddlylink` Eventos del mouse
3. ✅ `dom-data-tiddler-title` Obtenga el enlace que apunta a tiddler Titulo
4. ✅ uso state tiddler Almacene la vista previa actual tiddler
5. ✅ Reveal widget Muestre ventanas emergentes en ubicaciones apropiadas

#### 2.2 Plantilla de vista previa Tiddler

Crear `$:/plugins/yourname/link-preview/template`

```wikitext
title: $:/plugins/yourname/link-preview/template
type: text/vnd.tiddlywiki

<div class="preview-header">
  <h3><$link to=<<currentTiddler>>><$view field="title"/></$link></h3>
</div>

<div class="preview-meta">
  <$list filter="[<currentTiddler>has[subtitle]]" variable="ignore">
    <div class="preview-subtitle">
      <$view field="subtitle"/>
    </div>
  </$list>
  
  <div class="preview-info">
    <span class="preview-modified">
      Modified: <$view field="modified" format="relativedate"/>
    </span>
  </div>
</div>

<div class="preview-content">
  <$transclude mode="block" />
</div>

<div class="preview-tags">
  <$list filter="[<currentTiddler>tags[]]">
    <$link to=<<currentTiddler>>>
      <$macrocall $name="tag-pill" tag=<<currentTiddler>>/>
    </$link>
  </$list>
</div>
```

#### 2.3 estilo Tiddler

Crear `$:/plugins/yourname/link-preview/styles`

```css
title: $:/plugins/yourname/link-preview/styles
tags: $:/tags/Stylesheet
type: text/css

.link-preview-catcher {
  /* EventCatcher Los contenedores no afectan el diseno. */
  position: relative;
}

.link-preview-popup {
  /* Estilo basico de la ventana emergente. */
  z-index: 10000;
  max-width: 600px;
  min-width: 300px;
  font-size: 0.9em;
}

.link-preview-container {
  /* Efecto vidrio esmerilado */
  backdrop-filter: blur(10px) saturate(180%);
  background: rgba(255, 255, 255, 0.85);
  border: 1px solid rgba(0, 0, 0, 0.1);
  border-radius: 8px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.15);
  padding: 16px;
  max-height: 400px;
  overflow-y: auto;
}

/* Adaptacion del tema oscuro. */
html[data-theme="dark"] .link-preview-container {
  background: rgba(40, 40, 40, 0.85);
  border-color: rgba(255, 255, 255, 0.1);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.5);
}

.preview-header h3 {
  margin: 0 0 8px 0;
  font-size: 1.2em;
}

.preview-meta {
  margin-bottom: 12px;
  color: rgba(0, 0, 0, 0.6);
  font-size: 0.85em;
}

html[data-theme="dark"] .preview-meta {
  color: rgba(255, 255, 255, 0.6);
}

.preview-content {
  margin: 12px 0;
  line-height: 1.6;
  /* Limite el numero de lineas de contenido mostradas */
  display: -webkit-box;
  -webkit-line-clamp: 10;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.preview-tags {
  margin-top: 12px;
  padding-top: 8px;
  border-top: 1px solid rgba(0, 0, 0, 0.1);
}

html[data-theme="dark"] .preview-tags {
  border-top-color: rgba(255, 255, 255, 0.1);
}

/* efectos de animacion */
.link-preview-popup {
  animation: preview-fade-in 0.2s ease-out;
}

@keyframes preview-fade-in {
  from {
    opacity: 0;
    transform: translateY(-10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* Embellecimiento de la barra de desplazamiento */
.link-preview-container::-webkit-scrollbar {
  width: 6px;
}

.link-preview-container::-webkit-scrollbar-track {
  background: rgba(0, 0, 0, 0.05);
  border-radius: 3px;
}

.link-preview-container::-webkit-scrollbar-thumb {
  background: rgba(0, 0, 0, 0.2);
  border-radius: 3px;
}

.link-preview-container::-webkit-scrollbar-thumb:hover {
  background: rgba(0, 0, 0, 0.3);
}
```

---

## 3. Necesidad JS oportuno .tid + .meta Metodo de implementacion

Si es necesario JavaScript（Por ejemplo, si necesita funciones avanzadas como retardo y antivibracion), se recomienda utilizar `.ts` Archivo + `.ts.meta` manera。

### 3.1 TypeScript Archivos de modulo

Crear `link-hover.ts`

```typescript
/*\
title: $:/plugins/yourname/link-preview/link-hover.ts
type: application/javascript
module-type: startup

Link hover preview with debounce
\*/

export const name = "link-preview-hover";
export const platforms = ["browser"];
export const after = ["render"];
export const synchronous = true;

export function startup() {
  let timeout: number | null = null;
  const delay = 500; // 500ms Retraso
  
  // Escuche los eventos del mouse para todos los enlaces.
  document.addEventListener('mouseover', (event) => {
    const target = event.target as HTMLElement;
    const link = target.closest('.tc-tiddlylink');
    
    if (!link) return;
    
    // Borrar el temporizador anterior
    if (timeout) {
      clearTimeout(timeout);
    }
    
    // Visualizacion retrasada
    timeout = window.setTimeout(() => {
      const title = link.getAttribute('data-tiddler-title');
      if (!title) return;
      
      // Establecer estado
      $tw.wiki.setText('$:/state/link-preview/current', 'text', null, title);
      
      // Gatillo popup
      const rect = link.getBoundingClientRect();
      const coords = `[(${rect.left},${rect.bottom},${rect.width},0)]`;
      $tw.wiki.setText('$:/state/link-preview/popup', 'text', null, coords);
    }, delay);
  }, true);
  
  document.addEventListener('mouseout', (event) => {
    const target = event.target as HTMLElement;
    const link = target.closest('.tc-tiddlylink');
    
    if (!link) return;
    
    // Borrar temporizador
    if (timeout) {
      clearTimeout(timeout);
      timeout = null;
    }
    
    // Cierre retrasado
    setTimeout(() => {
      const popup = document.querySelector('.link-preview-popup:hover');
      if (!popup) {
        $tw.wiki.deleteTiddler('$:/state/link-preview/popup');
      }
    }, 100);
  }, true);
}
```

### 3.2 Meta Archivo

Crear `link-hover.ts.meta`

```
title: $:/plugins/yourname/link-preview/link-hover.ts
type: application/javascript
module-type: startup
```

---

## 4. Analisis comparativo

### 4.1 puro Wikitext Planificar

**Ventajas：**
- ✅ No se requiere escritura JavaScript
- ✅ Facil de mantener y depurar
- ✅ Uso completo TiddlyWiki Mecanismo nativo
- ✅ Buen desempeno (delegacion de eventos）
- ✅ Buena compatibilidad

**Desventajas：**
- ⚠️ No se puede lograr la visualizacion retrasada (activacion inmediata)）
- ⚠️ No se puede lograr la funcion antivibracion.
- ⚠️ No se puede implementar una logica compleja de movimiento del mouse

**Escenarios aplicables：**
- Requisitos simples de vista previa flotante
- No se requieren demoras ni interacciones complejas
- Quieres mantenerte puro wikitext Complementos

### 4.2 JavaScript Planificar

**Ventajas：**
- ✅ Se puede lograr una visualizacion retrasada
- ✅ Se puede lograr antivibracion/Acelerador
- ✅ Se puede implementar una logica de interaccion compleja
- ✅ Control mas detallado

**Desventajas：**
- ⚠️ Requiere redaccion y mantenimiento. JS 代码
- ⚠️ Puede haber problemas de rendimiento (necesita optimizacion)）
- ⚠️ La depuracion es relativamente compleja

**Escenarios aplicables：**
- La visualizacion debe retrasarse para evitar toques accidentales
- Requiere una logica compleja de interaccion con el mouse
- Altos requisitos en cuanto a la experiencia del usuario.

---

## 5. Plan final recomendado

### Sugerencias sobre la seleccion del plan.：

1. **Escenario simple (recomendado）**: uso **puro Wikitext + EventCatcher** Planificar
   - Sencillo de implementar y facil de mantener
   - Excelente desempeno
   - Cumplir con TiddlyWiki Filosofia

2. **Escenas complejas**: uso **Solucion hibrida**
   - Para infraestructura wikitext（PageTemplate + Reveal）
   - Para procesamiento de eventos TypeScript（Retraso, anti-vibracion）
   - Para renderizar plantillas wikitext

### Resumen de mejores practicas：

#### ✅ Practicas recomendadas：

1. uso `$:/tags/PageTemplate` Ajustar a nivel de pagina EventCatcher
2. uso `EventCatcher Widget` Manejo de eventos del mouse
3. uso `Reveal Widget` + `type="popup"` Mostrar ventana emergente
4. uso state tiddler Estado de gestion
5. uso CSS `backdrop-filter` Lograr el efecto de vidrio esmerilado
6. uso `tc-popup-keep` class Deja que la ventana emergente no se cierre al hacer clic dentro

#### ❌ Evita practicas：

1. No modifiques el nucleo globalmente link widget
2. No usar `$:/tags/RawMarkup` Inyectar globalmente JS
3. No vincule detectores de eventos independientes a cada enlace.
4. No usar `.js` Documentacion (usando `.ts` Mejor）
5. No olvides tratar los temas oscuros.

---

## 6. Estructura completa del complemento

```
$:/plugins/yourname/link-preview/
├── plugin.info                 # Metainformacion del complemento
├── page-template.tid          # PageTemplate (EventCatcher)
├── preview-template.tid       # Vista previa de plantillas de contenido
├── styles.tid                 # Estilo (efecto vidrio esmerilado）
├── config.tid                 # Interfaz de configuracion (opcional）
├── readme.tid                 # Documentacion
└── (Opcional) link-hover.ts       # JS Modulo (si necesita retraso y otras funciones）
    └── link-hover.ts.meta     # Meta Archivo
```

### plugin.info Ejemplo：

```json
{
  "title": "$:/plugins/yourname/link-preview",
  "description": "Modern link hover preview with glass effect",
  "author": "YourName",
  "version": "1.0.0",
  "core-version": ">=5.2.0",
  "plugin-type": "plugin",
  "list": "readme config"
}
```

---

## 7. Sugerencias para una mayor optimizacion

### 7.1 Optimizacion del rendimiento

1. **Truncamiento de contenido**: Limite la longitud del contenido de la vista previa en las plantillas
2. **Carga diferida**: Cargue el contenido completo solo cuando sea necesario
3. **Almacenamiento en cache**: uso state tiddler Contenido de vista previa de cache

### 7.2 Experiencia de usuario

1. **Animacion degradada**: uso CSS animation Visualizacion fluida
2. **Ajuste de ubicacion inteligente**: Ajuste la posicion de la ventana emergente de acuerdo con el limite de la ventana grafica.
3. **Teclas de acceso directo**: Apoyo ESC tecla para cerrar la vista previa
4. **Configurable**: Proporcione opciones de configuracion (tiempo de retraso, estilo, etc.)）

### 7.3 Accesibilidad

1. **ARIA Etiquetas**: Anade lo apropiado aria Propiedades
2. **Navegacion por teclado**: Admite operacion de teclado
3. **Lectores de pantalla**: Proporcionar texto descriptivo.

---

## 8. Recursos de referencia

### Documentos oficiales：

- [EventCatcherWidget](https://tiddlywiki.com/static/EventCatcherWidget.html)
- [RevealWidget](https://tiddlywiki.com/static/RevealWidget.html)
- [ActionPopupWidget](https://tiddlywiki.com/static/ActionPopupWidget.html)
- [ActionWidgets](https://tiddlywiki.com/static/ActionWidgets.html)
- [PopupMechanism](https://tiddlywiki.com/static/PopupMechanism.html)
- [SystemTags](https://tiddlywiki.com/static/SystemTags.html)

### Discusion comunitaria：

- [Talk TiddlyWiki - Preview Plugin](https://talk.tiddlywiki.org/)
- [GitHub - tobibeer/tw5-preview](https://github.com/tobibeer/tw5-preview)

---

## 9. Conclusion

Para los tiempos modernos TiddlyWiki5 Desarrollo de complementos，**Se recomienda utilizar puro Wikitext + EventCatcher Planificar**，Este es el mas apropiado TiddlyWiki Como se realiza la filosofia. Solo considere agregar funciones avanzadas como retardo y antivibracion si realmente las necesita. TypeScript Modulos。

Por uso legitimo EventCatcher、Reveal Widget y Action Widgets，Se puede implementar un complemento de vista previa de enlaces con funciones completas, excelente rendimiento y facil mantenimiento sin escribir ningun JavaScript 代码。
