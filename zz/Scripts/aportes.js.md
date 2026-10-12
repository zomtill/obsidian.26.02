const ramo = dv.current().ramo;

const evaluaciones = dv.pages("")
    .where(p => p.ramo === ramo)
    .where(p => p.tipo === "evaluacion");

const filas = [];

// ==========================================
// EVALUACIONES NORMALES
// ==========================================

for (const p of evaluaciones) {

    if (p.bloque != null) continue;

    if (
        p.nota == null ||
        p.ponderacion == null
    ) continue;

    const peso = Number(p.ponderacion);

    const aporte =
        Number(p.nota) * peso / 100;

    filas.push([
        p.tema ?? "Sin nombre",
        Number(p.nota).toFixed(2),
        `${peso.toFixed(2)}%`,
        aporte.toFixed(2)
    ]);
}


// ==========================================
// AGRUPAR BLOQUES
// ==========================================

const bloques = new Map();

for (const p of evaluaciones) {

    if (p.bloque == null) continue;

    const nombre = String(p.bloque);

    if (!bloques.has(nombre)) {
        bloques.set(nombre, []);
    }

    bloques.get(nombre).push(p);
}


// ==========================================
// PROCESAR BLOQUES
// ==========================================

for (const [nombre, grupo] of bloques) {

    const pesoBloque =
        Number(grupo[0].ponderacion_bloque);

    if (!pesoBloque) continue;


    // ======================================
    // BLOQUE DINÁMICO
    // ======================================

    const esDinamico = grupo.every(
        p => p.ponderacion == null
    );

    if (esDinamico) {

        const conNota = grupo.filter(
            p => p.nota != null
        );

        if (conNota.length === 0) continue;

        const promedio =
            conNota.reduce(
                (suma, p) => suma + Number(p.nota),
                0
            ) / conNota.length;

        const aporte =
            promedio * pesoBloque / 100;

        filas.push([
            `🗂️ ${nombre}`,
            promedio.toFixed(2),
            `${pesoBloque.toFixed(2)}%`,
            aporte.toFixed(2)
        ]);
    }


    // ======================================
    // BLOQUE DIVIDIDO
    // ======================================

    else {

        for (const p of grupo) {

            if (
                p.nota == null ||
                p.ponderacion == null
            ) continue;

            const peso =
                Number(p.ponderacion) *
                pesoBloque / 100;

            const aporte =
                Number(p.nota) * peso / 100;

            filas.push([
                `${nombre} - ${p.tema ?? "Parte"}`,
                Number(p.nota).toFixed(2),
                `${peso.toFixed(2)}%`,
                aporte.toFixed(2)
            ]);
        }
    }
}


dv.table(
    ["🫧 Evaluación / Bloque", "Nota", "🔷 Ponderación", "🔷 Aporte"],
    filas
);