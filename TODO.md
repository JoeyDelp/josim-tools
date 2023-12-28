Circuit netlist: Allow analysis or optimization of components in a specified subcircuit!

Margins: Option to output plot to a text file.

Yield: Add default values for **sd**.

Parameters: When not specified, get nominal values from the netlist file. Changes needed in:
* schema.py > MARGIN_PARAMETER
* schema.py > OPTIMIZE_PARAMETER
* schema.py > YIELD_PARAMETER
* configuration.py > def __init__(self, nominal: float, min_: Optional[float], max_: Optional[float]):
* configuration.py > def from_dict(value: Dict) -> "OptimizerParameterConfiguration":
* configuration.py > class MarginParameterConfiguration:
* configuration.py > class YieldParameterConfiguration:

CLI to allow specification of:
 * **num threads**, which is currently set automatically in tools.py as:
     num_threads = min(2 * len(margin_parameters), cpu_count())
 * **working directory**

Optimizer: Add **trend** option to print the iteration number and changed values of the Optimized Critical Margin on separate lines so that the trend is preserved.

Optimizer: Implement adjustable and testable parameters. (Why?)

Optimizer: Implement non-percentage based parameters. (Why?)
