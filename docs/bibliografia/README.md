# Biblioteca local (no versionada)

Carpeta para material bibliográfico de consulta en formato PDF/EPUB.

**No se sube a GitHub.** Los archivos dentro están excluidos en `.gitignore` por motivos de copyright. Solo esta carpeta y este `README.md` se versionan.

## Uso

- Deja aquí los PDFs/EPUBs de libros o artículos que quieras tener accesibles durante el desarrollo.
- Puedes referenciarlos en `docs/evidencia_cientifica.md` con cita completa (autor, año, página) sin necesidad de adjuntar el archivo al repo.
- En sesiones con Claude, los archivos de esta carpeta son legibles localmente para extraer datos, pero nunca se publican.

## Organización sugerida

```
bibliografia/
├── libros/
│   └── burke_practical_sports_nutrition.pdf
├── articulos/
│   └── thomas_2016_position_stand.pdf
└── posicionamientos/
    └── iom_2018_consensus.pdf
```
