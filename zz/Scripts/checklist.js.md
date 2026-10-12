const TIPOS_TAREA = [
    "tarea",
    "trabajo",
    "proyecto",
    "laboratorio",
    "estudio"
];

// =========================
// BUSCAR TAREAS
// =========================

const tareas = dv.pages("")
    .where(p =>
        TIPOS_TAREA.includes(
            String(p.tipo ?? "").toLowerCase()
        )
    )
    .where(p =>
        p.ramo != null &&
        String(p.ramo).trim() !== ""
    )
    .where(p =>
        String(p.estado ?? "").toLowerCase() !== "completado"
    )
    .where(p => {

        const tags = (p.tags ?? [])
            .map(t =>
                String(t)
                    .replace("#", "")
                    .toLowerCase()
            );

        return !tags.includes("completado");
    });

dv.header(3, "🌿 Mis tareas");

if (tareas.length === 0) {

    dv.paragraph("🪴 No tienes tareas pendientes.");

} else {

    // =========================
    // OBTENER TAGS
    // =========================

    const obtenerTags = (tarea) => {

        return (tarea.tags ?? [])
            .map(t =>
                String(t)
                    .replace("#", "")
                    .toLowerCase()
            );
    };

    // =========================
    // ORDENAR POR FECHA
    // =========================

    const tareasOrdenadas = [...tareas].sort((a, b) => {

        if (a.fecha && b.fecha) {
            return dv.date(a.fecha) - dv.date(b.fecha);
        }

        if (a.fecha) return -1;

        if (b.fecha) return 1;

        return String(a.tema ?? a.file.name)
            .localeCompare(
                String(b.tema ?? b.file.name)
            );
    });

    // =========================
    // CREAR TABLA
    // =========================

    const tabla = document.createElement("table");

    tabla.style.width = "100%";
    tabla.style.borderCollapse = "collapse";

    // =========================
    // ENCABEZADO
    // =========================

    const thead = document.createElement("thead");
    const encabezado = document.createElement("tr");

    const columnas = [
        "🌿",
        "🫧 Tarea",
        "🩵 Ramo",
        "🫧 Entrega",
        "🩵 Tags"
    ];

    for (const columna of columnas) {

        const th = document.createElement("th");

        th.innerText = columna;
        th.style.textAlign = "left";
        th.style.padding = "8px";

        encabezado.appendChild(th);
    }

    thead.appendChild(encabezado);
    tabla.appendChild(thead);

    // =========================
    // CUERPO
    // =========================

    const tbody = document.createElement("tbody");

    for (const tarea of tareasOrdenadas) {

        const fila = document.createElement("tr");

        // =========================
        // CHECKBOX
        // =========================

        const celdaCheckbox =
            document.createElement("td");

        celdaCheckbox.style.padding = "8px";

        const checkbox =
            document.createElement("input");

        checkbox.type = "checkbox";
        checkbox.style.cursor = "pointer";

        celdaCheckbox.appendChild(checkbox);

        // =========================
        // NOMBRE DE LA TAREA
        // =========================

        const celdaTarea =
            document.createElement("td");

        celdaTarea.style.padding = "8px";

        const enlace =
            document.createElement("a");

        enlace.innerText =
            tarea.tema ?? tarea.file.name;

        enlace.href = "#";
        enlace.style.cursor = "pointer";

        enlace.addEventListener("click", (evento) => {

            evento.preventDefault();

            app.workspace.openLinkText(
                tarea.file.path,
                tarea.file.path,
                false
            );
        });

        celdaTarea.appendChild(enlace);

        // =========================
        // RAMO
        // =========================

        const celdaRamo =
            document.createElement("td");

        celdaRamo.style.padding = "8px";

        celdaRamo.innerText =
            tarea.ramo ?? "-";

        // =========================
        // FECHA
        // =========================

        const celdaFecha =
            document.createElement("td");

        celdaFecha.style.padding = "8px";

        if (tarea.fecha) {

            celdaFecha.innerText =
                dv.date(tarea.fecha)
                    .toFormat("dd/MM/yyyy");

        } else {

            celdaFecha.innerText = "-";
        }

        // =========================
        // TAGS
        // =========================

        const celdaTags =
            document.createElement("td");

        celdaTags.style.padding = "8px";

        const tags = obtenerTags(tarea);

        const tagsMostrar = tags.filter(tag =>
            [
                "urgente",
                "importante",
                "pendiente",
                "en-progreso",
                "aplazar",
                "hoy",
                "esta-semana",
                "repasar",
                "practicar"
            ].includes(tag)
        );

        if (tagsMostrar.length > 0) {

            celdaTags.innerText =
                tagsMostrar
                    .map(tag => `#${tag}`)
                    .join(" ");

        } else {

            celdaTags.innerText = "-";
        }

        // =========================
        // AGREGAR FILA
        // =========================

        fila.appendChild(celdaCheckbox);
        fila.appendChild(celdaTarea);
        fila.appendChild(celdaRamo);
        fila.appendChild(celdaFecha);
        fila.appendChild(celdaTags);

        tbody.appendChild(fila);

        // =========================
        // MARCAR COMO COMPLETADA
        // =========================

        checkbox.addEventListener("change", async () => {

            if (!checkbox.checked) return;

            const archivo =
                app.vault.getAbstractFileByPath(
                    tarea.file.path
                );

            if (!archivo) return;

            await app.fileManager.processFrontMatter(
                archivo,
                (frontmatter) => {

                    // Estado
                    frontmatter.estado =
                        "completado";

                    // Obtener tags actuales
                    let tagsActuales =
                        frontmatter.tags ?? [];

                    if (!Array.isArray(tagsActuales)) {

                        tagsActuales = [
                            tagsActuales
                        ];
                    }

                    // Quitar #
                    tagsActuales =
                        tagsActuales.map(tag =>
                            String(tag)
                                .replace("#", "")
                        );

                    // Quitar estados anteriores
                    tagsActuales =
                        tagsActuales.filter(tag =>
                            ![
                                "pendiente",
                                "en-progreso"
                            ].includes(
                                tag.toLowerCase()
                            )
                        );

                    // Agregar completado
                    if (
                        !tagsActuales.some(tag =>
                            tag.toLowerCase() ===
                            "completado"
                        )
                    ) {

                        tagsActuales.push(
                            "completado"
                        );
                    }

                    frontmatter.tags =
                        tagsActuales;
                }
            );

            // Efecto visual
            enlace.style.textDecoration =
                "line-through";

            enlace.style.opacity =
                "0.5";

            // Eliminar de la tabla
            setTimeout(() => {
                fila.remove();
            }, 500);
        });
    }

    tabla.appendChild(tbody);

    // =========================
    // MOSTRAR TABLA
    // =========================

    dv.container.appendChild(tabla);
}