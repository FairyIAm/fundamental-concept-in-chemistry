# fundamental-concept-in-chemistry
balancing chemical equations. This program will allow you to input a chemical equation and it will output the balanced equation. The code uses the sympy library to solve the system of linear equations that represents the conservation of mass for each element.
pip install sympy
import sympy as sp

def parse_compound(compound):
    """Parses a chemical compound into its elements and their counts."""
    import re
    elements = re.findall(r'([A-Z][a-z]*)(\d*)', compound)
    return {el: int(num) if num else 1 for el, num in elements}

def parse_side(side):
    """Parses a side of the equation (either reactants or products)."""
    compounds = side.split(' + ')
    compound_dicts = [parse_compound(compound) for compound in compounds]
    all_elements = set(el for compound in compound_dicts for el in compound)
    return compounds, compound_dicts, all_elements

def balance_equation(equation):
    """Balances a chemical equation."""
    left_side, right_side = equation.split(' -> ')
    left_compounds, left_dicts, left_elements = parse_side(left_side)
    right_compounds, right_dicts, right_elements = parse_side(right_side)

    all_elements = left_elements.union(right_elements)
    compounds = left_compounds + right_compounds

    # Create the system of equations
    equations = []
    for element in all_elements:
        equation = sum(compound.get(element, 0) * sp.Symbol(f'x{i}')
                       for i, compound in enumerate(left_dicts)) - \
                   sum(compound.get(element, 0) * sp.Symbol(f'x{i + len(left_dicts)}')
                       for i, compound in enumerate(right_dicts))
        equations.append(equation)

    # Solve the system of equations
    symbols = [sp.Symbol(f'x{i}') for i in range(len(compounds))]
    solution = sp.solve(equations, symbols, dict=True)
    if not solution:
        return "No solution found. The equation might be impossible to balance."

    # Convert the solution to integers
    coeffs = [solution[0][symbol].as_numer_denom()[0] for symbol in symbols]
    lcm = sp.lcm([coeff.denominator for coeff in coeffs])
    coeffs = [coeff * lcm for coeff in coeffs]

    # Format the balanced equation
    balanced_left = ' + '.join(f'{coeff} {compound}' for coeff, compound in zip(coeffs[:len(left_compounds)], left_compounds))
    balanced_right = ' + '.join(f'{coeff} {compound}' for coeff, compound in zip(coeffs[len(left_compounds):], right_compounds))
    return f'{balanced_left} -> {balanced_right}'

# Example usage
equation = input("Enter a chemical equation to balance (e.g., H2 + O2 -> H2O): ")
balanced_equation = balance_equation(equation)
print("Balanced equation:", balanced_equation)
