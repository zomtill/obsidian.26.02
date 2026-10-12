// ==========================================
// PROMEDIO GENERAL PONDERADO POR CRÉDITOS
// ==========================================

const evaluaciones = dv.pages("")
    .where(p => p.tipo === "evaluacion");


// ==========================================
// AGRUPAR POR RAMO
// ==========================================

const ramos = new Map();

for (const p of evaluaciones) {

    if (p.ramo == null) continue;

    const nombre = String(p.ramo);

    if (!ramos.has(nombre)) {
        ramos.set(nombre, []);
    }

    ramos.get(nombre).push(p);
}


// ==========================================
// CALCULAR PROMEDIO DE CADA RAMO
// ==========================================

const filas = [];

let sumaPonderada = 0;
let totalCreditos = 0;


for (const [ramo, grupo] of ramos) {

    let pesoEvaluado = 0;
    let aporteActual = 0;


    // ======================================
    // EVALUACIONES NORMALES
    // ======================================

    for (const p of grupo) {

        if (p.bloque != null) continue;

        if (
            p.nota == null ||
            p.ponderacion == null
        ) continue;

        const peso = Number(p.ponderacion);

        pesoEvaluado += peso;

        aporteActual +=
            Number(p.nota) * peso / 100;
    }


    // ======================================
    // AGRUPAR BLOQUES
    // ======================================

    const bloques = new Map();

    for (const p of grupo) {

        if (p.bloque == null) continue;

        const nombreBloque = String(p.bloque);

        if (!bloques.has(nombreBloque)) {
            bloques.set(nombreBloque, []);
        }

        bloques.get(nombreBloque).push(p);
    }


    // ======================================
    // PROCESAR BLOQUES
    // ======================================

    for (const [nombreBloque, bloque] of bloques) {

        const pesoBloque =
            Number(bloque[0].ponderacion_bloque);

        if (!pesoBloque) continue;


        // ==================================
        // BLOQUE DINÁMICO
        // ==================================

        const esDinamico = bloque.every(
            p => p.ponderacion == null
        );

        if (esDinamico) {

            const conNota = bloque.filter(
                p => p.nota != null
            );

            if (conNota.length > 0) {

                const promedio =
                    conNota.reduce(
                        (suma, p) =>
                            suma + Number(p.nota),
                        0
                    ) / conNota.length;

                pesoEvaluado += pesoBloque;

                aporteActual +=
                    promedio * pesoBloque / 100;
            }
        }


        // ==================================
        // BLOQUE DIVIDIDO
        // ==================================

        else {

            for (const p of bloque) {

                if (
                    p.nota == null ||
                    p.ponderacion == null
                ) continue;

                const peso =
                    Number(p.ponderacion) *
                    pesoBloque / 100;

                pesoEvaluado += peso;

                aporteActual +=
                    Number(p.nota) * peso / 100;
            }
        }
    }


    // ======================================
    // PROMEDIO DEL RAMO
    // ======================================

    if (pesoEvaluado > 0) {

        const promedio =
            aporteActual /
            (pesoEvaluado / 100);


        // ==================================
        // BUSCAR PANEL PRINCIPAL DEL RAMO
        // ==================================

        const paginaRamo = dv.pages("")
            .where(p =>
                p.tipo === "sub_panel" &&
                String(p.ramo).trim() === String(ramo).trim()
            )
            .first();


        // ==================================
        // LEER CRÉDITOS
        // ==================================

        let creditos = 0;

        if (paginaRamo && paginaRamo.creditos != null) {

            creditos = Number(
                String(paginaRamo.creditos)
                    .replace(",", ".")
                    .replace(/[^\d.]/g, "")
            );
        }


        // ==================================
        // CREAR ENLACE
        // ==================================

        const enlace = paginaRamo
            ? paginaRamo.file.link
            : ramo;


        filas.push([
            enlace,
            promedio.toFixed(2),
            `${creditos} SCT`,
            `${pesoEvaluado.toFixed(1)}%`
        ]);


        // ==================================
        // PROMEDIO PONDERADO
        // ==================================

        if (creditos > 0) {

            sumaPonderada +=
                promedio * creditos;

            totalCreditos += creditos;
        }
    }
}


// ==========================================
// ORDENAR RAMOS
// ==========================================

filas.sort((a, b) =>
    String(a[0]).localeCompare(String(b[0]))
);


// ==========================================
// PROMEDIO GENERAL
// ==========================================

const promedioGeneral =
    totalCreditos > 0
        ? (sumaPonderada / totalCreditos).toFixed(2)
        : "Sin créditos";


// ==========================================
// MOSTRAR TABLA
// ==========================================

dv.table(
    [
        "🩵 Ramo",
        "🩵 Promedio",
        "🩵 Créditos",
        "🌿 Evaluado"
    ],
    filas
);


// ==========================================
// MOSTRAR PROMEDIO GENERAL
// ==========================================

dv.paragraph(
    `🩵 **Promedio general ponderado:** ${promedioGeneral}`
);