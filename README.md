# sonnet-elevation

**Protocolo de elevación para modelos LLM: cómo hacer que un modelo menor opere cerca del nivel de un modelo frontier comprando profundidad con cómputo de inferencia.**

Autor: [Andrea Di Mare](https://github.com/dimeazza) · 2026

---

## El problema

Los modelos frontier (Claude Fable 5 y similares) son caros y de acceso limitado. Los modelos de trabajo diario (Claude Sonnet y similares) son rápidos y baratos, pero pierden en las tareas que más importan: decisiones con trade-offs, arquitectura, análisis de riesgo, verificación de fallos no obvios.

La capacidad bruta no se transfiere por prompt. Pero cuatro cosas sí:

| Palanca | Qué transfiere | Mecanismo |
|---|---|---|
| **GEN×3** | Profundidad de búsqueda | Generar 3 soluciones con supuestos divergentes ANTES de evaluar ninguna (best-of-N manual) |
| **Verificador adversarial** | Detección de defectos | Verificar es más fácil que generar: checks falsables escritos antes de mirar las soluciones |
| **Memoria externa** | Contexto extendido | Protocolo STATE.md + map-reduce para aproximar ventanas de contexto grandes |
| **Cartuchos de juicio** | Criterio del modelo frontier | Juicio pre-computado por el modelo grande sobre TUS dominios, que el modelo menor solo tiene que APLICAR |

Y una contramedida obligatoria: **regla de escalado**. El mayor riesgo del protocolo es producir respuestas con la *forma* de un modelo frontier sin su fondo. El skill obliga al modelo a declarar "esto excede mi verificación" ante 6 triggers concretos, en vez de fingir.

## Qué hay en este repo

```
skill/
  SKILL.md                      # El protocolo completo (formato Claude Skills)
  references/
    plantilla-state.md          # Memoria externa para tareas multi-sesión
    plantilla-cartucho.md       # Estructura de un cartucho de juicio destilado
    prompt-destilado.md         # El prompt para generar cartuchos con un modelo frontier
```

Los cartuchos son personales por diseño: contienen el juicio del modelo grande sobre tus sistemas concretos. Este repo incluye la plantilla y el prompt generador; tus cartuchos los generas tú y viven en tu máquina.

## Instalación (Claude)

**Claude Code:**
```bash
git clone https://github.com/dimeazza/sonnet-elevation.git
cp -r sonnet-elevation/skill ~/.claude/skills/sonnet-elevacion
```

**Claude.ai:** empaqueta la carpeta `skill/` como ZIP con extensión `.skill` y súbela en Ajustes → Capacidades → Skills.

## Uso

1. Genera tus cartuchos: abre una sesión con el modelo frontier, usa `references/prompt-destilado.md` con el contexto real de tu dominio, guarda el resultado en `references/cartucho-<dominio>.md`.
2. Trabaja normalmente con el modelo menor. El protocolo se activa en tareas complejas o escribiendo **"ELEVA"**.
3. Cuando el modelo declare `⚠️ ESCALADO RECOMENDADO`, esa pregunta va al modelo frontier — es la señal de que el protocolo llegó a su techo honestamente.
4. Regenera los cartuchos tras cada hito mayor o ~3 meses: el juicio destilado caduca.

## Resultados esperados (honestos)

Medido en mi propio flujo de trabajo: ~65–70% de similitud operacional con el modelo frontier en tareas cubiertas por cartuchos, ~55–65% en ejecución verificable sin cartucho, y ~20–30% en insight novedoso cross-domain — que es exactamente donde el protocolo escala en vez de fingir. No es magia: es disciplina de proceso + juicio pre-computado + honestidad sobre los límites.

## Licencia

MIT — ver [LICENSE](LICENSE).
