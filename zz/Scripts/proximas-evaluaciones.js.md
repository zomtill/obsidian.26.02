// ==========================================
// PRÓXIMAS EVALUACIONES
// ==========================================

const hoy = dv.date("today");

const evaluaciones = dv.pages("")
    .where(p => p.tipo === "evaluacion")
    .where(p => p.fecha != null)
    .where(p => p.nota == null)
    .where(p => dv.date(p.fecha) >= hoy)
    .sort(p => dv.date(p.fecha), "asc");


// ==========================================
// TABLA
// ==========================================

const filas = evaluaciones.map(p => {

    const fecha = dv.date(p.fecha);

    return [
        p.ramo ?? "Sin ramo",
        p.file.link,
        fecha.toFormat("dd/MM/yyyy")
    ];
});


if (filas.length === 0) {

    dv.paragraph("🪴 No tienes evaluaciones próximas.");

} else {

    dv.table(
        [
            "🩵 Ramo",
            "🫧 Evaluación",
            "🫧 Fecha"
        ],
        filas
    );
}

