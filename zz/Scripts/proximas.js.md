const ramo = dv.current().ramo;
const hoy = dv.date("today");

const evaluaciones = dv.pages("")
    .where(p => p.ramo === ramo)
    .where(p => p.tipo === "evaluacion");

const filas = [];


// ==========================================
// PROCESAR EVALUACIONES
// ==========================================

for (const p of evaluaciones) {

    // Ya tiene nota
    if (p.nota != null) continue;

    // No tiene fecha
    if (p.fecha == null) continue;

    // La fecha ya pasó
    if (dv.date(p.fecha) < hoy) continue;


    // ======================================
    // EVALUACIÓN NORMAL
    // ======================================

    if (p.bloque == null) {

        if (p.ponderacion == null) continue;

        filas.push([
            p.fecha,
            p.tema ?? "Sin nombre",
            `${Number(p.ponderacion).toFixed(2)}%`
        ]);

        continue;
    }


    // ======================================
    // EVALUACIÓN CON BLOQUE
    // ======================================

    const pesoBloque =
        Number(p.ponderacion_bloque);

    if (!pesoBloque) continue;


    // BLOQUE DINÁMICO

    if (p.ponderacion == null) {

        filas.push([
            p.fecha,
            p.tema ?? "Sin nombre",
            `${p.bloque} (${pesoBloque}%)`
        ]);
    }


    // BLOQUE DIVIDIDO

    else {

        const peso =
            Number(p.ponderacion) *
            pesoBloque / 100;

        filas.push([
            p.fecha,
            p.tema ?? "Sin nombre",
            `${peso.toFixed(2)}%`
        ]);
    }
}


// ==========================================
// ORDENAR POR FECHA
// ==========================================

filas.sort((a, b) =>
    dv.date(a[0]) - dv.date(b[0])
);


dv.table(
    ["🫧 Fecha", "🫧 Evaluación", "🔷 Ponderación"],
    filas
);