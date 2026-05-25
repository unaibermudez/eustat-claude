# Manejo de errores HTTP en Inkudea

## Regla principal: toast vs. navegación a error page

| Situación | Comportamiento |
|---|---|
| El error deja la página **sin datos críticos e irrecuperable** (ej. fallo de carga de estructura de la propia página) | Navegar a `pathNames.error` con título y mensaje |
| El error no impide que la página siga siendo funcional (tabla no actualizada, exportación fallida, filtros incompletos) | Toast de error — nunca navegar |

```
¿La pantalla se queda inconsistente sin esos datos?
  SÍ → navigate(pathNames.error)
  NO → showNotification(..., NotificationType.error)
```

---

## Errores de thunks Redux — automáticos via middleware

Los thunks creados con `createAsyncThunk` y `{ serializeError: serializeAxiosError }` **NO necesitan manejo manual de error**: el `notification-middleware` (`src/config/notification-middleware.ts`) intercepta todas las acciones rechazadas y muestra un toast automáticamente.

```typescript
// Correcto: el middleware gestiona el toast
dispatch(fetchProvincias());

// Incorrecto: doble toast (middleware + manual) y navegación innecesaria
dispatch(fetchProvincias()).then((result) => {
  if (fetchProvincias.rejected.match(result)) {
    setFetchFailed(true); // ← nunca hacer esto si sólo cabe un toast
    showNotification(...); // ← tampoco, el middleware ya lo hace
  }
});
```

El patrón obligatorio en todos los thunks:

```typescript
export const fetchAlgo = createAsyncThunk(
  "slice/fetchAlgo",
  async (params) => {
    const response = await api.get("/endpoint");
    return response.data;
  },
  { serializeError: serializeAxiosError }
);
```

---

## Errores de llamadas directas a `api` (sin Redux)

Las llamadas `api.get/post/put/delete` fuera de un thunk **no pasan por el middleware**. Hay que notificar manualmente:

```typescript
import { showNotification } from "../../components/Notification/Notification";
import { NotificationType } from "../../enums/Notification";

try {
  const response = await api.get("/endpoint");
  // éxito...
} catch (err: any) {
  if (err?.code !== "ERR_CANCELED") {
    // No navegar — mostrar toast salvo que la pantalla quede inconsistente
    showNotification({
      message: t("mi-componente.error.carga", "Error al cargar los datos"),
      type: NotificationType.error,
    });
  }
  throw err; // re-lanzar si el llamador necesita saberlo (ej. CustomTable)
}
```

El chequeo `err?.code !== "ERR_CANCELED"` es **obligatorio** para ignorar los errores de peticiones abortadas (AbortController / signal).

---

## Navegación a la pantalla de error

Usar **solo** cuando la página se queda en un estado irrecuperable:

```typescript
const navigate = useNavigate();

// Ejemplo: fallo que hace inutilizable la página completa
const title = encodeURIComponent(t("mi-componente.error.titulo"));
const msg   = encodeURIComponent(t("mi-componente.error.carga"));
navigate(`${pathNames.error}?title=${title}&msg=${msg}`);
```

No usar `setFetchFailed` + `useEffect` para navegar: es más frágil y puede provocar el overlay rojo de webpack dev si la navegación ocurre en medio de ciclos async. Llamar `navigate(...)` directamente en el catch.

---

## Acciones de guardado / mutación con dispatch

El middleware muestra toasts automáticos en error. El componente sí debe gestionar el estado de loading y el toast de éxito (que el middleware no genera por defecto):

```typescript
const handleGuardar = async () => {
  setIsSaving(true);
  const result = await dispatch(updateAlgo(payload));
  setIsSaving(false); // siempre, antes del if/else

  if (updateAlgo.fulfilled.match(result)) {
    showNotification({ message: t("...guardado"), type: NotificationType.success });
    // cerrar offcanvas, refrescar tabla, etc.
  }
  // Si es rejected: el middleware ya mostró el toast de error → no mostrar otro
};
```

**Nunca llamar a `showNotification` de error manualmente después de un dispatch si el thunk tiene `serializeError: serializeAxiosError`**, para evitar doble toast.

---

## Claves i18n para mensajes de error

Cada componente debe tener sus propias claves en `locales/{es,eu,en}.json`:

```json
{
  "mi-componente": {
    "error": {
      "carga":    "Error al cargar los datos",
      "guardar":  "Error al guardar",
      "exportar": "Error al exportar los datos"
    }
  }
}
```

Si el componente usa el sistema de locales de `src/locales/` (no la carpeta local `locales/`), añadir al fichero correspondiente en `src/locales/es/`, `eu/` y `en/`.

---

## Resumen rápido

```
Thunk Redux con serializeAxiosError
  → middleware gestiona el toast automáticamente
  → no añadir manejo manual de error

api.get directo (no thunk)
  → catch manual + showNotification si err.code !== "ERR_CANCELED"
  → throw err si CustomTable u otro llamador necesita saberlo

¿Navegar a error page?
  → Solo si la pantalla queda inconsistente / irrecuperable
  → navigate(...) directo en el catch, no setFetchFailed+useEffect
```
