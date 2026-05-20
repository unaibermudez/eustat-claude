# Claude Skills para EUSTAT

Este repositorio contiene **skills personalizados de Claude** diseñados para automatizar y acelerar el desarrollo de aplicaciones frontend en EUSTAT. Estos skills documentan convenciones, patrones y buenas prácticas del proyecto que Claude utiliza para generar código consistente y de alta calidad.

## 📦 Instalación

Para que estos skills funcionen en el frontend de EUSTAT, **debes copiar la carpeta `.claude` a la raíz de tu proyecto frontend**:

```bash
# Desde la raíz del proyecto frontend
cp -r /ruta/a/eustat-claude/.claude ./
```

Una vez copiada, la carpeta `.claude` estará disponible cuando trabajes con Claude Code en ese proyecto. Claude utilizará automáticamente los skills y configuraciones definidos aquí.

> **Nota:** Este repositorio complementa el archivo `CLAUDE.md` del proyecto frontend. La guía completa de desarrollo está en `CLAUDE.md` — esta carpeta `.claude` contiene skills adicionales que Claude utiliza para mejorar su asistencia en tareas específicas.

## 🎯 Skills Disponibles

- **`BOTONES.md`** — Convenciones de estilos para botones de acción (colores, hover, padding)
- **`FORMULARIOS_UNSAVED_CHANGES.md`** — Hook `useUnsavedChangesWarning` para proteger cambios sin guardar

Consulta cada archivo en `.claude/SKILLS/` para detalles completos y ejemplos de código.

## 🚀 Cómo Funciona

1. Cuando trabajas en el frontend del proyecto con Claude Code, la carpeta `.claude` se detecta automáticamente
2. Claude carga los skills y utiliza esa información para:
   - Generar código que sigue las convenciones del proyecto
   - Sugerir componentes y patrones consistentes
   - Evitar errores comunes en estilos y comportamientos

## 💡 Próximos Pasos

Cuando agregues nuevas convenciones, patrones reutilizables o descubras mejores prácticas:

1. Crea un nuevo archivo `.md` en la carpeta `SKILLS/`
2. Documenta la convención o patrón con ejemplos de código
3. Comparte la actualización con el equipo

---

**Mantenido por:** Unai Bermudez  
**Última actualización:** Mayo 2026
