# Reflection Report on The Beaver's Choice Paper Company Multi-Agent System Project

## Architecture and Decision Process

The workflow diagram shows five agents: one orchestrator and four specialized workers. The orchestrator receives each dated customer request, delegates work to the relevant agents, and combines their findings into a customer-facing response. This design separates the main business decisions by responsibility while keeping one agent accountable for the final response.

The Inventory Agent checks catalog items, available quantities, and shortages using the inventory tools. The Quote Agent looks for similar historical requests and calculates catalog prices with quantity-based discounts. The Order Fulfillment Agent estimates supplier delivery and records a sale only when the requested quantity is available. The Finance Agent provides cash and financial snapshots to support internal review. The workers return their findings to the orchestrator, which is responsible for communicating the outcome.

I chose this architecture because inventory availability, price estimation, fulfillment timing, and financial reporting require different data and decision rules. Separating them makes responsibilities and tool access easier to understand and test. The orchestrator provides a single entry point for requests and coordinates the sequence: check inventory, prepare a quote, check delivery feasibility, record an eligible order, and summarize the result. The implementation uses `smolagents` `ToolCallingAgent` instances and configures the four workers as managed agents of the orchestrator.

## Evaluation Results

The saved `test_results.csv` contains 20 rows with 20 distinct request IDs, covering all requests in `quote_requests_sample.csv`. Six requests are recorded as confirmed orders: requests 1, 4, 5, 6, 10, and 12. The other 14 are not confirmed, and each response gives one or more reasons, including insufficient stock, unsupported catalog items or sizes, incompatible unit descriptions, or delivery estimates later than the requested deadline.

Cash changes between five adjacent result rows, meeting the evaluation target of at least three cash-balance changes. The six confirmed orders also exceed the target of three fulfilled requests. These changes are consistent with orders being recorded as sales, while requests that fail the complete-order checks are not represented as fulfilled. The dataset includes a range of outcomes: for example, request 1 is confirmed with itemized prices and a 5% discount, while request 3 is declined because A4 stock is short, A3 is not cataloged, and printer paper was requested in reams although the catalog tracks units per sheet or item. Request 9 is declined in part because estimated delivery is after its April 10 deadline.

The saved customer responses explain applied discounts and give concrete reasons for declined orders. They do not disclose the cash balance or inventory valuation, which keeps the customer-facing content focused on the order. The division between quote, inventory, fulfillment, and finance responsibilities also makes it possible to review the basis for an outcome instead of relying on an unexplained final decision.

## Further Improvements

1. **Make evaluation deterministic and repeatable.** Persist the inventory seed and coverage used for a run, and add an evaluation mode that initializes a known stock fixture. Record structured status fields such as `confirmed`, `declined`, `reason`, and `delivery_date` alongside the response. This would make counts and cash changes easy to audit and reproduce.

2. **Enforce fulfillment checks in code, not only agent instructions.** Validate every line item’s stock and delivery deadline before recording any sale, and make the transaction operation atomic for a multi-item order. This prevents partial writes if one item is unavailable and ensures an order is never described as confirmed unless all required checks pass.
