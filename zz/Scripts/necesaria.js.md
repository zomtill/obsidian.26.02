const ramo = dv.current().ramo;

const objetivos = [4.0, 5.0, 6.0];

const evaluaciones = dv.pages("")
    .where(p => p.ramo === ramo)
    .where(p => p.tipo === "evaluacion");

let aporteActual = 0;
let pesoPendiente = 0;


// ==========================================
// EVALUACIONES NORMALES
// ==========================================

for (const p of evaluaciones) {

    if (p.bloque != null) continue;

    if (p.ponderacion == null) continue;

    const peso = Number(p.ponderacion);

    if (p.nota != null) {

        aporteActual +=
            Number(p.nota) * peso / 100;

    } else {

        pesoPendiente += peso;
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

        const pendientes = grupo.filter(
            p => p.nota == null
        );


        if (conNota.length > 0) {

            const promedio =
                conNota.reduce(
                    (suma, p) => suma + Number(p.nota),
                    0
                ) / conNota.length;

            aporteActual +=
                promedio * pesoBloque / 100;

        } else if (pendientes.length > 0) {

            pesoPendiente += pesoBloque;
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

                aporteActual +=
                    Number(p.nota) * peso / 100;

            } else {

                pesoPendiente += peso;
            }
        }
    }
}


// ==========================================
// RESULTADO
// ==========================================

if (pesoPendiente <= 0) {

    dv.paragraph(
        "🪴 No quedan evaluaciones pendientes conocidas."
    );

} else {

    const filas = objetivos.map(objetivo => {

        const necesario =
            (objetivo - aporteActual) /
            (pesoPendiente / 100);

        let resultado;

        if (necesario <= 1) {

            resultado = "1,0 o más";

        } else if (necesario > 7) {

            resultado = "🫧 No es posible";

        } else {

            resultado = necesario.toFixed(2);
        }

        return [
            objetivo.toFixed(1),
            resultado
        ];
    });


    dv.table(
        ["🌿 Promedio final objetivo", "🫧 Nota necesaria"],
        filas
    );
}