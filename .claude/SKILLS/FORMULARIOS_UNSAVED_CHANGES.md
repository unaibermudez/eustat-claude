# Advertencia de cambios sin guardar en formularios

**Regla:** Todo formulario que pueda cerrarse sin guardar (offcanvas, panel lateral, navegación entre secciones) **debe** usar `useUnsavedChangesWarning` para proteger los datos del usuario.

---

## Cómo funciona el componente

**Ubicación:** `src/components/useUnsavedChangesWarning/useUnsavedChangesWarning.tsx`

El hook recibe:
- `hasUnsavedChanges: boolean` — normalmente `isDirty` de `react-hook-form`
- `onResetChanges?: () => void` — callback para resetear el estado del formulario cuando el usuario confirma salir

Devuelve:
- `showWarningAndExecute(action)` — muestra el modal y ejecuta `action` si el usuario confirma
- `executeImmediately(action)` — ejecuta `action` sin modal (cuando no hay cambios)
- `WarningModal` — componente que renderiza el modal de advertencia; **siempre debe incluirse en el JSX**

Además intercepta automáticamente:
- Clics en enlaces `<a href>` de la misma origen cuando hay cambios pendientes
- El evento `beforeunload` del navegador (cierre de pestaña / recarga)

---

## Patrón de uso — formulario en el mismo componente

```tsx
import { useUnsavedChangesWarning } from "path/to/useUnsavedChangesWarning";

const { showWarningAndExecute, executeImmediately, WarningModal } = useUnsavedChangesWarning({
  hasUnsavedChanges: isDirty,           // de react-hook-form
  onResetChanges: () => reset(),        // reset del formulario
});

// En cualquier acción que saque al usuario del formulario:
const handleClose = () => {
  const close = () => setOpen(false);
  if (isDirty) {
    showWarningAndExecute(close);
    return;
  }
  executeImmediately(close);
};

// En el JSX (al final del return):
return (
  <div>
    {/* ... */}
    <WarningModal />
  </div>
);
```

---

## Patrón de uso — formulario en componente hijo (offcanvas)

Cuando el formulario (`HelpForm`, etc.) vive dentro de un offcanvas gestionado por un componente padre (`HelpTab`), el estado `isDirty` se eleva al padre mediante la prop `onUnsavedChangesChange`.

**En el padre (ej. `HelpTab`):**

```tsx
const [formHasUnsavedChanges, setFormHasUnsavedChanges] = useState(false);

const { showWarningAndExecute, executeImmediately, WarningModal } = useUnsavedChangesWarning({
  hasUnsavedChanges: formHasUnsavedChanges,
  onResetChanges: () => setFormHasUnsavedChanges(false),
});

const handleCloseOffcanvas = () => {
  if (loading) return;
  const close = () => setOffCanvas(false);
  if (formHasUnsavedChanges) {
    showWarningAndExecute(close);
    return;
  }
  executeImmediately(close);
};

// En el JSX del hijo, propagar y actualizar el estado local:
<HelpForm
  onUnsavedChangesChange={(hasChanges) => {
    setFormHasUnsavedChanges(hasChanges);
    onUnsavedChangesChange?.(hasChanges);  // seguir propagando al abuelo si hace falta
  }}
/>

// Al final del JSX del padre:
<WarningModal />
```

**En el hijo (ej. `HelpForm`):**

```tsx
// useEffect que notifica al padre cuando isDirty cambia (ya existente):
useEffect(() => {
  onUnsavedChangesChange?.(isDirty);
}, [isDirty, onUnsavedChangesChange]);
```

---

## Ejemplos existentes en el proyecto

| Archivo | Tipo |
|---|---|
| `src/pages/GestionEncuestas/ConfiguracionEncuesta/tabs/InformationTab.tsx` | Formulario en el mismo componente, navegación entre secciones |
| `src/pages/GestionEncuestas/ConfiguracionEncuesta/tabs/cuestionariosTab/DatosCuestionario.tsx` | Formulario en el mismo componente, navegación entre secciones |
| `src/pages/GestionEncuestas/ConfiguracionEncuesta/tabs/helpTab/HelpTab.tsx` | Formulario hijo en offcanvas |
