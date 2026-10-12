// ==========================================
// 🫧 PANEL DE ESTUDIO
// ==========================================
//
// Detecta automáticamente:
//
// .../Estudio/Temas/
// .../Estudio/Repaso/
// .../Estudio/Errores/
//
// Estados:
//
// # estudio/necesito-repasar
// # estudio/en-proceso
// # estudio/dominado
//
// Propiedad opcional:
//
// proximo-repaso: 2026-09-28
//
// ==========================================


// ==========================================
// 🩵 TAGS
// ==========================================

const TAG_REPASAR =
    "estudio/necesito-repasar";

const TAG_PROCESO =
    "estudio/en-proceso";

const TAG_DOMINADO =
    "estudio/dominado";


// ==========================================
// 🔹 OBTENER TAGS
// ==========================================

const obtenerTags = (pagina) => {

    return (pagina.tags ?? [])
        .flatMap(tag => {

            if (Array.isArray(tag)) {
                return tag;
            }

            return [tag];

        })
        .map(tag =>
            String(tag)
                .replace(/^#/, "")
                .toLowerCase()
                .trim()
        );

};


// ==========================================
// 🩵 OBTENER RAMO
// ==========================================

const obtenerRamo = (pagina) => {

    if (pagina.ramo) {
        return String(pagina.ramo);
    }

    const partes =
        String(pagina.file.path)
            .split("/");

    const indiceEstudio =
        partes.lastIndexOf("Estudio");

    if (indiceEstudio > 0) {
        return partes[indiceEstudio - 1];
    }

    return "-";

};


// ==========================================
// 🗂️ TODAS LAS NOTAS
// ==========================================

const todasLasNotas = dv.pages("");


// ==========================================
// 🩵 TEMAS
// ==========================================

const temas = todasLasNotas
    .where(p => {

        const ruta =
            String(p.file.path)
                .toLowerCase();

        return ruta.includes(
            "/estudio/temas/"
        );

    });


// ==========================================
// ♻️ REPASOS
// ==========================================

const repasos = todasLasNotas
    .where(p => {

        const ruta =
            String(p.file.path)
                .toLowerCase();

        return ruta.includes(
            "/estudio/repaso/"
        );

    });


// ==========================================
// 🫧 ERRORES
// ==========================================

const errores = todasLasNotas
    .where(p => {

        const ruta =
            String(p.file.path)
                .toLowerCase();

        return ruta.includes(
            "/estudio/errores/"
        );

    });


// ==========================================
// 🩵 ESTADOS
// ==========================================

const necesitoRepasar =
    temas.where(p =>
        obtenerTags(p)
            .includes(TAG_REPASAR)
    );


const enProceso =
    temas.where(p =>
        obtenerTags(p)
            .includes(TAG_PROCESO)
    );


const dominado =
    temas.where(p =>
        obtenerTags(p)
            .includes(TAG_DOMINADO)
    );


// ==========================================
// 🫧 FECHA ACTUAL
// ==========================================

const hoy =
    dv.date(
        window.moment().format("YYYY-MM-DD")
    );


// ==========================================
// 🫧 REPASAR HOY
// ==========================================

const repasarHoy =
    temas.where(p => {

        if (!p.proximoRepaso) {
            return false;
        }

        const fecha =
            dv.date(p.proximoRepaso);

        if (!fecha) {
            return false;
        }

        return fecha
            .toFormat("yyyy-MM-dd")
            === hoy.toFormat("yyyy-MM-dd");

    });


// ==========================================
// 🌱 REPASOS ATRASADOS
// ==========================================

const repasosAtrasados =
    temas.where(p => {

        if (!p.proximoRepaso) {
            return false;
        }

        const fecha =
            dv.date(p.proximoRepaso);

        if (!fecha) {
            return false;
        }

        return fecha < hoy;

    });


// ==========================================
// 🫧 ESTILOS
// ==========================================

const estilo =
    document.createElement("style");

estilo.textContent = `

.estudio-panel {

    width: 100%;

}


/* =========================================
   TÍTULO
   ========================================= */

.estudio-titulo {

    font-size: 1.8em;

    font-weight: 700;

    margin-bottom: 5px;

}


.estudio-subtitulo {

    opacity: 0.7;

    margin-bottom: 20px;

}


/* =========================================
   ESTADOS
   ========================================= */

.estudio-grid {

    display: grid;

    grid-template-columns:
        repeat(3, 1fr);

    gap: 15px;

    margin-bottom: 20px;

}


.estudio-card {

    background:
        var(--background-secondary);

    border:
        1px solid
        var(--background-modifier-border);

    border-radius: 14px;

    padding: 16px;

}


.estudio-card h3 {

    margin-top: 0;

    margin-bottom: 5px;

}


.estudio-contador {

    font-size: 0.85em;

    opacity: 0.6;

    margin-bottom: 12px;

}


/* =========================================
   TEMAS
   ========================================= */

.estudio-tema {

    padding: 9px;

    margin-bottom: 7px;

    border-radius: 8px;

    background:
        var(--background-primary);

}


.estudio-tema a {

    text-decoration: none;

    cursor: pointer;

}


.estudio-ramo {

    font-size: 0.8em;

    opacity: 0.6;

    margin-top: 3px;

}


/* =========================================
   SECCIONES
   ========================================= */

.estudio-seccion {

    margin-top: 18px;

    padding: 16px;

    background:
        var(--background-secondary);

    border:
        1px solid
        var(--background-modifier-border);

    border-radius: 14px;

}


.estudio-seccion h3 {

    margin-top: 0;

    margin-bottom: 12px;

}


/* =========================================
   LISTAS
   ========================================= */

.estudio-lista {

    display: grid;

    grid-template-columns:
        repeat(2, 1fr);

    gap: 8px;

}


.estudio-item {

    padding: 10px;

    border-radius: 8px;

    background:
        var(--background-primary);

}


.estudio-item a {

    text-decoration: none;

    cursor: pointer;

}


/* =========================================
   REPASOS
   ========================================= */

.estudio-fecha {

    font-size: 0.8em;

    opacity: 0.6;

    margin-top: 3px;

}


/* =========================================
   VACÍO
   ========================================= */

.estudio-vacio {

    opacity: 0.5;

    font-style: italic;

}


/* =========================================
   RAMOS
   ========================================= */

.estudio-ramos {

    display: grid;

    grid-template-columns:
        repeat(2, 1fr);

    gap: 12px;

}


.estudio-ramo-card {

    padding: 14px;

    border-radius: 12px;

    background:
        var(--background-primary);

}


.estudio-ramo-card h4 {

    margin-top: 0;

    margin-bottom: 10px;

}


.estudio-ramo-stats {

    font-size: 0.85em;

    opacity: 0.8;

    line-height: 1.7;

}


/* =========================================
   MÓVIL
   ========================================= */

@media (max-width: 800px) {

    .estudio-grid {

        grid-template-columns: 1fr;

    }

    .estudio-lista {

        grid-template-columns: 1fr;

    }

    .estudio-ramos {

        grid-template-columns: 1fr;

    }

}

`;

dv.container.appendChild(estilo);


// ==========================================
// 🔷 PANEL
// ==========================================

const panel =
    document.createElement("div");

panel.className =
    "estudio-panel";


// ==========================================
// 🫧 ESTRUCTURA
// ==========================================

panel.innerHTML = `

<div class="estudio-titulo">

    🫧 Panel de Estudio

</div>


<div class="estudio-subtitulo">

    Organiza tus contenidos, repasos y errores.

</div>


<!-- ======================================
     ESTADOS
     ====================================== -->

<div class="estudio-grid">


    <div class="estudio-card">

        <h3>
            🩵 Necesito repasar
        </h3>

        <div class="estudio-contador">

            ${necesitoRepasar.length}
            tema(s)

        </div>

        <div id="lista-repasar"></div>

    </div>


    <div class="estudio-card">

        <h3>
            🫧 En proceso
        </h3>

        <div class="estudio-contador">

            ${enProceso.length}
            tema(s)

        </div>

        <div id="lista-proceso"></div>

    </div>


    <div class="estudio-card">

        <h3>
            🌿 Dominado
        </h3>

        <div class="estudio-contador">

            ${dominado.length}
            tema(s)

        </div>

        <div id="lista-dominado"></div>

    </div>

</div>


<!-- ======================================
     🫧 REPASAR HOY
     ====================================== -->

<div class="estudio-seccion">

    <h3>
        🫧 Repasar hoy
    </h3>

    <div
        id="lista-hoy"
        class="estudio-lista">
    </div>

</div>


<!-- ======================================
     🌱 ATRASADOS
     ====================================== -->

<div class="estudio-seccion">

    <h3>
        🌱 Repasos atrasados
    </h3>

    <div
        id="lista-atrasados"
        class="estudio-lista">
    </div>

</div>


<!-- ======================================
     🩵 POR RAMO
     ====================================== -->

<div class="estudio-seccion">

    <h3>
        🩵 Por ramo
    </h3>

    <div
        id="lista-ramos"
        class="estudio-ramos">
    </div>

</div>


<!-- ======================================
     🫧 ERRORES
     ====================================== -->

<div class="estudio-seccion">

    <h3>
        🫧 Errores recientes
    </h3>

    <div
        id="lista-errores"
        class="estudio-lista">
    </div>

</div>


<!-- ======================================
     ♻️ REPASOS
     ====================================== -->

<div class="estudio-seccion">

    <h3>
        ♻️ Repasos
    </h3>

    <div
        id="lista-repasos"
        class="estudio-lista">
    </div>

</div>

`;


// ==========================================
// 🔹 CREAR ENLACE
// ==========================================

function crearEnlace(pagina) {

    const enlace =
        document.createElement("a");

    enlace.textContent =
        pagina.tema ??
        pagina.file.name;

    enlace.href = "#";

    enlace.addEventListener(
        "click",
        evento => {

            evento.preventDefault();

            app.workspace.openLinkText(

                pagina.file.path,

                pagina.file.path,

                false

            );

        }
    );

    return enlace;

}


// ==========================================
// 🫧 MOSTRAR TEMAS
// ==========================================

function mostrarTemas(lista, id) {

    const contenedor =
        panel.querySelector(`#${id}`);


    if (lista.length === 0) {

        contenedor.innerHTML = `

            <div class="estudio-vacio">

                No hay temas aquí 🪴

            </div>

        `;

        return;

    }


    lista.forEach(pagina => {

        const tema =
            document.createElement("div");

        tema.className =
            "estudio-tema";


        tema.appendChild(
            crearEnlace(pagina)
        );


        const ramo =
            document.createElement("div");

        ramo.className =
            "estudio-ramo";

        ramo.textContent =
            `🩵 ${obtenerRamo(pagina)}`;

        tema.appendChild(ramo);


        contenedor.appendChild(tema);

    });

}


// ==========================================
// 🫧 MOSTRAR REPASOS
// ==========================================

function mostrarRepasos(lista, id, atrasado = false) {

    const contenedor =
        panel.querySelector(`#${id}`);


    if (lista.length === 0) {

        contenedor.innerHTML = `

            <div class="estudio-vacio">

                ${atrasado
                    ? "No tienes repasos atrasados 🪴"
                    : "No tienes repasos para hoy 🪴"}

            </div>

        `;

        return;

    }


    lista.forEach(pagina => {

        const item =
            document.createElement("div");

        item.className =
            "estudio-item";


        item.appendChild(
            crearEnlace(pagina)
        );


        const ramo =
            document.createElement("div");

        ramo.className =
            "estudio-ramo";

        ramo.textContent =
            `🩵 ${obtenerRamo(pagina)}`;

        item.appendChild(ramo);


        if (pagina.proximoRepaso) {

            const fecha =
                document.createElement("div");

            fecha.className =
                "estudio-fecha";

            fecha.textContent =
                `🫧 ${dv.date(
                    pagina.proximoRepaso
                ).toFormat("dd/MM/yyyy")}`;

            item.appendChild(fecha);

        }


        contenedor.appendChild(item);

    });

}


// ==========================================
// 🩵 CREAR RESUMEN POR RAMO
// ==========================================

function crearResumenRamos() {

    const contenedor =
        panel.querySelector(
            "#lista-ramos"
        );


    const grupos = {};


    temas.forEach(pagina => {

        const ramo =
            obtenerRamo(pagina);


        if (!grupos[ramo]) {

            grupos[ramo] = {

                repasar: 0,

                proceso: 0,

                dominado: 0,

                total: 0

            };

        }


        grupos[ramo].total++;


        const tags =
            obtenerTags(pagina);


        if (tags.includes(TAG_REPASAR)) {

            grupos[ramo].repasar++;

        }


        if (tags.includes(TAG_PROCESO)) {

            grupos[ramo].proceso++;

        }


        if (tags.includes(TAG_DOMINADO)) {

            grupos[ramo].dominado++;

        }

    });


    const nombres =
        Object.keys(grupos)
            .sort();


    if (nombres.length === 0) {

        contenedor.innerHTML = `

            <div class="estudio-vacio">

                No hay ramos todavía 🪴

            </div>

        `;

        return;

    }


    nombres.forEach(ramo => {

        const datos =
            grupos[ramo];


        const tarjeta =
            document.createElement("div");

        tarjeta.className =
            "estudio-ramo-card";


        tarjeta.innerHTML = `

            <h4>
                🩵 ${ramo}
            </h4>

            <div class="estudio-ramo-stats">

                🩵 ${datos.repasar}
                por repasar

                <br>

                🫧 ${datos.proceso}
                en proceso

                <br>

                🌿 ${datos.dominado}
                dominados

                <br>

                🌊 ${datos.total}
                temas

            </div>

        `;


        contenedor.appendChild(tarjeta);

    });

}


// ==========================================
// 🫧 MOSTRAR ERRORES
// ==========================================

function mostrarErrores() {

    const contenedor =
        panel.querySelector(
            "#lista-errores"
        );


    if (errores.length === 0) {

        contenedor.innerHTML = `

            <div class="estudio-vacio">

                No hay errores registrados 🪴

            </div>

        `;

        return;

    }


    const erroresOrdenados =
        [...errores]
            .sort(
                (a, b) =>
                    b.file.mtime -
                    a.file.mtime
            );


    erroresOrdenados.forEach(pagina => {

        const item =
            document.createElement("div");

        item.className =
            "estudio-item";


        item.appendChild(
            crearEnlace(pagina)
        );


        const ramo =
            document.createElement("div");

        ramo.className =
            "estudio-ramo";

        ramo.textContent =
            `🩵 ${obtenerRamo(pagina)}`;

        item.appendChild(ramo);


        contenedor.appendChild(item);

    });

}


// ==========================================
// 🦋 MOSTRAR TODO
// ==========================================

mostrarTemas(
    necesitoRepasar,
    "lista-repasar"
);


mostrarTemas(
    enProceso,
    "lista-proceso"
);


mostrarTemas(
    dominado,
    "lista-dominado"
);


mostrarRepasos(
    repasarHoy,
    "lista-hoy"
);


mostrarRepasos(
    repasosAtrasados,
    "lista-atrasados",
    true
);


crearResumenRamos();


mostrarErrores();


mostrarRepasos(
    repasos,
    "lista-repasos"
);


// ==========================================
// 🦋 MOSTRAR PANEL
// ==========================================

dv.container.appendChild(panel);