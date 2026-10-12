const ramo = dv.current().ramo;

const evaluaciones = dv.pages("")
    .where(p => p.ramo === ramo)
    .where(p => p.tipo === "evaluacion");

let pesoEvaluado = 0;
let aporteActual = 0;


// ==========================================
// EVALUACIONES NORMALES
// ==========================================

for (const p of evaluaciones) {

    // Las evaluaciones con bloque se procesan aparte
    if (p.bloque != null) continue;

    if (p.ponderacion == null) continue;

    const peso = Number(p.ponderacion);

    if (p.nota != null) {

        pesoEvaluado += peso;

        aporteActual +=
            Number(p.nota) * peso / 100;
    }
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

        if (conNota.length > 0) {

            const promedio =
                conNota.reduce(
                    (suma, p) => suma + Number(p.nota),
                    0
                ) / conNota.length;

            pesoEvaluado += pesoBloque;

            aporteActual +=
                promedio * pesoBloque / 100;
        }
    }


    // ======================================
    // BLOQUE DIVIDIDO
    // ======================================

    else {

        for (const p of grupo) {

            if (p.ponderacion == null) continue;

            const peso =
                Number(p.ponderacion) *
                pesoBloque / 100;

            if (p.nota != null) {

                pesoEvaluado += peso;

                aporteActual +=
                    Number(p.nota) * peso / 100;
            }
        }
    }
}


// ==========================================
// RESULTADO
// ==========================================

let promedio = "Sin notas";

if (pesoEvaluado > 0) {

    promedio =
        (aporteActual / (pesoEvaluado / 100))
        .toFixed(2);
}

const pesoRestante =
    Math.max(0, 100 - pesoEvaluado);


dv.table(
    ["🩵 Promedio actual", "🌿 Evaluado", "🔹 Restante"],
    [[
        promedio,
        `${pesoEvaluado.toFixed(1)}%`,
        `${pesoRestante.toFixed(1)}%`
    ]]
);
