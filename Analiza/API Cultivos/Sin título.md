---
tags:
  - Agiles
  - Amazon
---
Tengo un servicio de impresión en Python que maneja impresoras térmicas y de etiquetas. El problema específico es que **el contenido de los stickers se imprime fuera del área de la etiqueta** (muy hacia la izquierda, parcialmente fuera del papel). Necesito mover todo el contenido hacia la derecha (aumentar coordenadas X).

## 🔍 INFORMACIÓN TÉCNICA:

- **Lenguaje:** Python 3
- **Protocolo:** TSPL (para impresoras de etiquetas)
- **Tamaño etiqueta:** 320x200 dots (40mm x 25mm a 203dpi)
- **Impresora:** 4BARCODE 4B-2054L (TSC compatible)
- **Sistema:** Windows con win32print

## 📸 SÍNTOMA VISUAL:

El contenido se imprime parcialmente fuera del borde izquierdo de la etiqueta. Necesito mover todo hacia la derecha para que quede dentro del área imprimible.

## 🔧 CÓDIGO RELEVANTE:

python

```python
def convertir_escpos_a_tspl(self, contenido):
    """Convierte comandos ESC/POS a TSPL para impresoras de etiquetas"""
    
    tspl = "SIZE 400, 240\n"
    tspl += "GAP 24, 0\n"
    tspl += "DIRECTION 1\n"
    tspl += "CLS\n\n"
    
    # Variables de estado
    y_position = 30    # ← POSICIÓN Y INICIAL
    font_size = "2"
    bold = False
    align = "left"
    
    # Procesar línea por línea
    lineas = contenido.split('<NL>')
    
    for linea in lineas:
        # [código de procesamiento de formato...]
        
        # Si hay texto, imprimirlo
        texto = texto.strip()
        if texto:
            texto = self.limpiar_texto_utf8(texto)
            
            x_pos = 50          # ← POSICIÓN X POR DEFECTO
            if align == "center":
                x_pos = 200     # ← POSICIÓN X CENTRO
            elif align == "right":
                x_pos = 350     # ← POSICIÓN X DERECHA
            
            # Agregar comando TEXT
            tspl += f'TEXT {x_pos},{y_position},"{font_size}",0,1,1,"{texto}"\n'
            y_position += 30    # ← INCREMENTO Y
    
    tspl += "\nPRINT 1\n"
    return tspl

def imprimir_etiqueta(contenido, impresora):
    # [código para detectar si es lista o string...]
    
    if isinstance(contenido, list):
        # Procesa datos JSON como lista
        for idx, item in enumerate(contenido):
            # Área
            if area:
                tspl_commands.append(f'TEXT 25,15,"1",0,1,1,"{area}"')
            
            # Nombre  
            if nombre:
                tspl_commands.append(f'TEXT 25,40,"2",0,1,1,"{nombre_corto}"')
            
            # Edad
            if edad_formateada:
                tspl_commands.append(f'TEXT 25,65,"2",0,1,1,"Edad: {edad_formateada}"')
            
            # Género
            if genero:
                tspl_commands.append(f'TEXT 160,65,"2",0,1,1,"Genero: {genero}"')
                
            # Código de barras
            if orden:
                tspl_commands.append(f'BARCODE 25,100,"128",45,1,0,2,2,"{orden}"')

def imprimir_ter(contenido, impresora):
    # Detecta tipo de impresora
    es_impresora_etiquetas = any(keyword in impresora.upper() for keyword in 
        ['ETIQUETA', '4BARCODE', 'LDT114', '3NSTAR', 'TSC', 'ZEBRA', 'GODEX'])
    
    if es_impresora_etiquetas:
        # USA convertir_escpos_a_tspl() ← AQUÍ ESTÁ EL PROBLEMA
        new_contenido = self.convertir_escpos_a_tspl(contenido)
    else:
        # USA procesamiento ESC/POS normal
        new_contenido = reemplazar(contenido)
```

## ❓ PREGUNTAS DIAGNÓSTICAS PRECISAS:

**1. IDENTIFICAR FUNCIÓN ACTIVA:**

- ¿Mi impresora "4BARCODE" está siendo detectada como impresora de etiquetas?
- ¿Esto significa que usa `convertir_escpos_a_tspl()` en lugar de `imprimir_etiqueta()`?
- ¿Cómo puedo confirmar qué función está ejecutándose?

**2. COORDENADAS X PROBLEMÁTICAS:**

- En `convertir_escpos_a_tspl()`, ¿los valores `x_pos = 50`, `x_pos = 200`, `x_pos = 350` están causando que el contenido se imprima fuera del área?
- ¿Qué valores de X serían apropiados para una etiqueta de 320 dots de ancho?
- ¿Debería aumentar estos valores X para mover el contenido hacia la derecha?

**3. VALORES ESPECÍFICOS A CAMBIAR:**

- Para mover contenido hacia la derecha, ¿debo cambiar `x_pos = 50` a qué valor?
- ¿Los valores `x_pos = 200` (centro) y `x_pos = 350` (derecha) también necesitan ajuste?
- ¿El `y_position = 30` inicial y el incremento `y_position += 30` están bien?

**4. PROCESO DE DATOS:**

- ¿Mi contenido llega como string con etiquetas `<NL>` o como lista JSON?
- ¿Cómo determino cuál de las dos funciones está procesando mis datos?
- ¿Debo modificar ambas funciones por seguridad?

## 🎯 LO QUE YA INTENTÉ:

- Modifiqué los valores X en `imprimir_etiqueta()` de 25 a 80, pero no hubo cambios
- Reinicié el servicio después de los cambios
- El contenido sigue imprimiéndose en la misma posición incorrecta

## 🚀 LO QUE NECESITO:

**Respuesta específica con:**

1. **Confirmación:** ¿Qué función está ejecutándose realmente?
2. **Valores exactos:** ¿Qué números cambiar en qué líneas?
3. **Código corregido:** Los valores X correctos para mover contenido hacia la derecha
4. **Método de verificación:** ¿Cómo confirmar que los cambios funcionaron?

## 📝 FORMATO DE RESPUESTA DESEADO:

```
FUNCIÓN ACTIVA: [nombre_de_función]
CAMBIOS REQUERIDOS:
- Línea X: cambiar [valor_actual] por [valor_nuevo]
- Línea Y: cambiar [valor_actual] por [valor_nuevo]
CÓDIGO CORREGIDO: [código_específico]
```

---

**¿Puedes analizar este código y darme los valores exactos de coordenadas X que debo cambiar para mover el contenido hacia la derecha en la etiqueta?**

