import json
import os
from typing import List, Dict, Any

from src.engine.types import Action, InputItem, Policy, PolicyResult
from src.engine.loader import load_policies
from src.engine.matcher import match_policies
from src.engine.resolver import resolve_policy, resolve_conflict, apply_action, HIERARCHY

def main():
    # Paths
    base_dir = os.path.dirname(os.path.abspath(__file__))
    # Data is located in src/data
    data_dir = os.path.join(base_dir, "src", "data")
    
    input_path = os.path.join(data_dir, "input.json")
    policies_path = os.path.join(data_dir, "policies.json")
    output_path = os.path.join(data_dir, "output.json")
    
    # Load Data
    try:
        policies, default_action_str = load_policies(policies_path)
        with open(input_path, 'r') as f:
            input_data = json.load(f)
    except Exception as e:
        print(f"Error loading data: {e}")
        return

    default_action = Action[default_action_str.upper()]
    results = []

    for item in input_data:
        input_item = InputItem(
            id=item["id"],
            risk=item.get("risk", "unknown"),
            output=item["output"],
            confidence=item["confidence"]
        )
        
        # Match Policies
        matched_policies = match_policies(input_item, policies)
        
        final_decision: Action = default_action
        final_reason: str = f"No policy matched risk '{input_item.risk}'. Default action applied."
        applied_policies_ids: List[str] = []
        
        if matched_policies:
            policy_results = []
            for policy in matched_policies:
                res = resolve_policy(policy, input_item.confidence)
                policy_results.append(res)
                
            # Resolve Conflict
            if policy_results:
                final_result = resolve_conflict(policy_results)
                if final_result:
                    final_decision = final_result.action
                    final_reason = final_result.reason
                    applied_policies_ids = [r.policy_id for r in policy_results]
        
        # Apply Action
        final_output_text = apply_action(final_decision, input_item.output)
        
        # Construct Result
        result_entry = {
            "id": input_item.id,
            "decision": final_decision.name.lower(),
            "applied_policies": applied_policies_ids,
            "final_output": final_output_text,
            "reason": final_reason
        }
        results.append(result_entry)
        
    # Write Output
    with open(output_path, 'w') as f:
        json.dump(results, f, indent=2)
        
    print(f"Processed {len(results)} items. Output written to {output_path}")

if __name__ == "__main__":
    main()
