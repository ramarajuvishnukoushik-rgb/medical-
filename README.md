# Guardrail Engine

This project implements a deterministic Guardrail Engine that processes user inputs against a set of safety policies. It matches input risks to policies, resolves conflicts based on a strict hierarchy, and determines the final action (Allow, Sanitize, Escalate, Block).

## How to Run

1.  **Prerequisites**: Python 3.8+
2.  **Clone the repository**:
    ```bash
    git clone <repository_url>
    cd guardrail-engine
    ```
3.  **Run the engine**:
    ```bash
    python3 main.py
    ```
4.  **Check Output**:
    The results will be generated in `src/data/output.json`.

## Project Structure

-   `main.py`: Entry point for the application.
-   `src/engine/`: Core logic modules (loader, matcher, resolver, types).
-   `src/data/`: Input data (`input.json`), policy definitions (`policies.json`), and results (`output.json`).

## Assumptions

1.  **Input Format**: inputs are provided in a JSON list format with `id`, `risk`, `confidence`, and `output` fields.
2.  **Policy Definitions**: Policies define a `risk` type, `allowed_actions`, and a `min_confidence` threshold.
3.  **Hierarchy**: The action hierarchy is strictly defined as `Block > Escalate > Sanitize > Allow`.
4.  **Escalation**: If a policy matches but confidence is below the threshold, the action is automatically escalated to the next verified restrictive level (as per `resolver.py` logic).
5.  **Default Action**: If no policy matches an input's risk, the default action specified in `policies.json` is applied.

## Tradeoffs

1.  **Determinism vs. Flexibility**: The engine prioritizes deterministic behavior (strict hierarchy, consistent tie-breaking) over complex, context-aware decision making. This ensures predictability but limits nuance.
2.  **Performance**: The current implementation loads policies and processes inputs sequentially. For extremely large datasets, parallel processing or streaming inputs would be more efficient.
3.  **Hardcoded Hierarchy**: The action hierarchy is hardcoded in `resolver.py`. Making this configuration-driven would increase flexibility but add complexity.
4.  **JSON Storage**: Using JSON files for data and policies is simple for this exercise but would likely be replaced by a database in a production environment for better scalability and query capabilities.
