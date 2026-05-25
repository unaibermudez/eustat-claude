# Convenciones de botones principales

Todos los botones de acción principal de esta aplicación siguen un único estilo. Usa siempre la clase `btn` de Bootstrap junto con la clase semántica del contexto y aplica los siguientes estilos SCSS. Las variables están en `src/styles/variables.scss`.

```scss
.btn.{nombre-clase} {
  background-color: $primary;   // #013161
  color: $white;
  border: none;
  border-radius: 0px;
  padding: 10px 20px;
  font-size: 0.95rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;

  &:hover {
    background-color: #c69507;  // amarillo dorado
    color: $primary;            // texto azul oscuro en hover
  }
}
```

## Reglas generales

- **Nunca** uses `box-shadow` en el hover; el cambio de color es suficiente.
- **Nunca** mantengas el mismo `background-color` en hover que en estado normal.
- Los botones destructivos (eliminar) no siguen esta convención; usan `$danger`.
- Para botones centrados en formularios (submit en offcanvas/modal) añade `display: block; margin: 40px auto 0; min-width: 240px; text-align: center;` y el estado `&:disabled { opacity: 0.6; cursor: not-allowed; }`, manteniendo el mismo estilo visual.
