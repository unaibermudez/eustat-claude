# CLAUDE.md — inkudea-canal-web/frontend

Guía de desarrollo para el frontend de Inkudea (EUSTAT). Lee esto antes de tocar cualquier archivo.

---

## Stack tecnológico

| Categoría | Tecnología |
|---|---|
| Framework | React 18 + TypeScript 5 (strict) |
| Bundler | Webpack 5 (no Vite) |
| Estado global | Redux Toolkit 2 |
| Formularios | React Hook Form 7 |
| Routing | React Router 6 |
| UI | Bootstrap 5 + Reactstrap 9 + MUI 7 |
| Estilos | SCSS (Sass) + variables globales |
| HTTP | Axios (instancia configurada en `src/axios/api.ts`) |
| i18n | i18next + react-i18next |
| Testing | Jest 29 + Testing Library + ts-jest |
| Linting | ESLint 9 (flat config) + Prettier |

---

## Comandos

```bash
npm start          # Dev server en puerto 9000
npm run build      # Build de producción
npm test           # Tests (Jest)
npm run lint       # ESLint check
npm run lint:fix   # ESLint con auto-fix
npm run full-check # clean + lint + test + build
```

El proxy de desarrollo redirige `/api` → `http://localhost:6060/inkudea/api/v1`.

---

## Estructura de directorios

```
src/
├── axios/              # Instancia Axios + interceptores
├── components/         # Componentes reutilizables
├── config/             # Store Redux, constants, pathNames, PrivateRoute
├── enums/              # Enums TypeScript del dominio
├── hooks/              # Custom hooks
├── layout/             # Layout principal (Header, Footer, contenido)
├── locales/            # Configuración i18n + JSONs de traducción
├── pages/              # Componentes de página
├── slices/             # Redux slices (estado + thunks)
├── styles/             # Estilos globales y variables SCSS
├── types/              # Interfaces TypeScript del dominio
├── utils/              # Helpers: validators, storage, common
├── App.tsx             # Root component (sesión, i18n init)
└── AppRoutes.tsx       # Definición de rutas
```

---

## Convenciones de código

### Estructura de un componente

Cada componente vive en su propia carpeta con esta estructura:

```
ComponentName/
├── ComponentName.tsx       # Componente funcional
├── ComponentName.scss      # Estilos (importados dentro del .tsx)
├── ComponentName.test.tsx  # Tests
└── locales/                # JSONs de i18n locales (se fusionan en build)
    ├── es.json
    └── eu.json
```

### Nombrado

- Componentes y tipos: `PascalCase`
- Variables, funciones, hooks: `camelCase`
- Archivos de componente: `PascalCase.tsx`
- Archivos de utilidad/hook: `camelCase.ts`
- Interfaces de props: `interface ComponentNameProps { ... }`
- No hay alias `@` — usar rutas relativas estándar

### Imports

ESLint impone este orden (separados por línea en blanco):

1. `builtin` (node)
2. `external` (npm: react, redux…)
3. `internal` (src/)
4. `parent` (`../`)
5. `sibling` (`./`)

### TypeScript

- `strict: true` activo — no usar `any` si se puede evitar
- Usar siempre `useAppSelector` y `useAppDispatch` (tipados), nunca los genéricos de Redux
- Tipos del dominio en `src/types/`, enums en `src/enums/`

---

## Estado global (Redux)

### Slices

| Slice | Ruta | Responsabilidad |
|---|---|---|
| `authorization` | `src/slices/authorizationSlice.ts` | Sesión, tokens JWT, cuenta |
| `locale` | `src/slices/localeSlice.ts` | Idioma activo |
| `encuestas` | `src/slices/encuestasSlice.ts` | Encuestas, cuestionarios, loading |

### Thunks async

Siempre usar `createAsyncThunk` con `serializeAxiosError`:

```typescript
import { serializeAxiosError } from "../utils/reducer.utils";

export const fetchData = createAsyncThunk(
  "slice/action",
  async (params: Params) => {
    const response = await api.get("/endpoint");
    return response.data;
  },
  { serializeError: serializeAxiosError }
);
```

### Leer y despachar

```typescript
const dispatch = useAppDispatch();
const { loading, data } = useAppSelector((state) => state.encuestas);
```

---

## Formularios

Usar siempre **React Hook Form**. Validadores reutilizables en `src/utils/validators.ts`:

```typescript
validateEmail(value, message)
validateTelefono(value, message)
validateMaxLength(value, max, message)
validateAllowedValues(value, allowed[], message)
```

Patrón estándar con `FormProvider` cuando hay subcomponentes:

```typescript
const formMethods = useForm<FormValues>({ mode: "onChange" });
const { register, handleSubmit, formState: { errors, isDirty } } = formMethods;

return (
  <FormProvider {...formMethods}>
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* subcomponentes usan useFormContext() */}
    </form>
  </FormProvider>
);
```

### Advertencia de cambios sin guardar — OBLIGATORIO

**Todo formulario que pueda cerrarse sin guardar debe usar `useUnsavedChangesWarning`.**

Ver `.claude/SKILLS/FORMULARIOS_UNSAVED_CHANGES.md` para el patrón detallado. En resumen:

```typescript
import { useUnsavedChangesWarning } from "path/to/useUnsavedChangesWarning";

const { showWarningAndExecute, executeImmediately, WarningModal } = useUnsavedChangesWarning({
  hasUnsavedChanges: isDirty,
  onResetChanges: () => reset(),
});

// Acción que saca al usuario del formulario:
const handleClose = () => {
  const close = () => setOpen(false);
  isDirty ? showWarningAndExecute(close) : executeImmediately(close);
};

// En el JSX:
return <div>...<WarningModal /></div>;
```

Cuando el formulario es un componente hijo (offcanvas), elevar `isDirty` al padre con `onUnsavedChangesChange` y gestionar el hook en el padre.

---

## Routing

Rutas definidas en `src/config/pathNames.ts`. Usar siempre ese enum en vez de strings hardcodeados:

```typescript
import pathNames from "../config/pathNames";
navigate(pathNames.gestionEncuestas);
```

Las rutas privadas se protegen con `<PrivateRoute>`. Soporta `hasAnyAuthorities` para RBAC.

---

## Internacionalización (i18n)

- Tres idiomas: `es`, `eu`, `en`
- Los JSONs locales de cada componente (`locales/es.json`) se fusionan automáticamente en build con los de `src/locales/`
- En producción el archivo final es `/locales/{lng}/app.json`

```typescript
const { t } = useTranslation();
t("common.save")                           // clave simple
t("common.validators.errorMaxLength", { maxLength: 10 })  // con parámetros
```

Para el idioma actual del store:

```typescript
const currentLocale = useAppSelector((state) => state.locale.currentLocale); // 'es' | 'eu' | 'en'
```

### Regla obligatoria: traducir todos los literales

**Nunca** dejar strings hardcodeados visibles al usuario en JSX ni en props de texto. Esto incluye:
- Texto de botones, labels, títulos, mensajes
- `placeholder`, `alt`, `aria-label`
- Mensajes de error y validación
- Textos de notificaciones / toasts

Siempre usar `t("clave", "fallback")` y añadir la clave en los tres JSONs (`es`, `eu`, `en`).

Si el componente no tiene carpeta `locales/`, crearla siguiendo la estructura estándar:

```
ComponentName/
└── locales/
    ├── es.json
    ├── eu.json
    └── en.json
```

El webpack recoge automáticamente cualquier `src/components/**/locales/{lang}.json` y lo fusiona en `app.json`. **No hace falta configuración adicional**, pero sí es necesario reiniciar el dev server si se añaden archivos nuevos durante una sesión activa.

---

## HTTP / API

La instancia de Axios está en `src/axios/api.ts`. Usa siempre esa instancia, nunca `axios` directamente:

```typescript
import api from "../../axios/api";
const response = await api.get("/Encuestas");
```

Los interceptores manejan automáticamente:
- Inyección del token JWT en headers
- Respuestas 401 → `clearAuthentication` (cierre de sesión)
- Timeout de 60 s
- Refresh de token SSO (Inksarrera)

---

## Estilos

- Variables globales en `src/styles/variables.scss` — usar siempre variables, nunca colores hardcodeados
- Colores principales: `$primary` (#013161 azul EUSTAT), `$white`
- Hover de botones: fondo `#c69507` (amarillo dorado), texto `$primary`
- Ver `.claude/SKILLS/BOTONES.md` para la convención completa de botones de acción

Cada componente importa su propio `.scss`. No hay CSS-in-JS.

---

## Testing

```typescript
// Wrapper con providers en src/__tests__/test-utils.tsx
import { render, screen } from "../../__tests__/test-utils";

// Mocks disponibles:
// - react-i18next → devuelve la clave como texto
// - axios → jest.mock manual en cada test
// - Redux store → renderWithProviders() de test-utils
```

Los tests de componentes van junto al componente (`ComponentName.test.tsx`). Los tests de utilidades van en `src/utils/__tests__/`.

---

## Notificaciones (toasts)

```typescript
import { toast } from "react-toastify";

toast.success(t("common.saved"));
toast.error(t("common.error-guardar"));
```

En componentes con offcanvas, usar un `ToastContainer` local con `containerId` para evitar conflictos con el global:

```tsx
<ToastContainer containerId="offcanvas-toast" position="top-right" autoClose={3000} style={{ zIndex: 99999 }} />
toast.error(message, { containerId: "offcanvas-toast" });
```

---

## Autenticación y sesión

- SSO vía Inksarrera (IdP custom). El login redirige al IdP externo.
- Tokens JWT en `localStorage` (claves en `src/config/constants.ts`)
- No manipular tokens directamente — usar las acciones del `authorizationSlice`
- `PrivateRoute` gestiona la redirección si el token ha caducado

---

## Skills de referencia

Documentación de convenciones del proyecto en `.claude/SKILLS/`:

| Archivo | Contenido |
|---|---|
| `BOTONES.md` | Estilos y clases CSS para botones de acción |
| `FORMULARIOS_UNSAVED_CHANGES.md` | Patrón `useUnsavedChangesWarning` en formularios |
