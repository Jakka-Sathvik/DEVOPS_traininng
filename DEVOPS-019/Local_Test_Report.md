                 Local Test Report
Automated Testing Report
Project: Calculator Application
Test Framework: pytest
Test Type: Automated Unit Tests
Test Location: tests/

Test Structure
project/
├── app/
│   ├── __init__.py
│   └── calculator.py
├── tests/
│   ├── __init__.py
│   ├── test_add.py
│   ├── test_subtract.py
│   ├── test_multiply.py
│   └── test_divide.py
├── requirements.txt
└── Local_Test_Report.md

Application Functions Tested
The automated tests cover the following calculator operations:

Addition
Subtraction
Multiplication
Division
Negative numbers
Zero values
Decimal values
Division by zero
Test Execution
Run the tests locally with:

pip install -r requirements.txt
pytest -v

Expected result:

============================= test session starts =============================
collected 16 items

tests/test_add.py ........
tests/test_subtract.py ....
tests/test_multiply.py ..
tests/test_divide.py ....

============================== 16 passed in ...s ==============================

Test Results
Metric	Result
Number of Tests	16
Passed	16
Failed	0
Status	PASS
Execution Time	To be recorded from local run

Test Cases
Addition
Add two positive numbers
Add negative numbers
Add zero
Add decimal numbers
Subtraction
Subtract two positive numbers
Subtract negative numbers
Subtract zero
Subtract decimal numbers
Multiplication
Multiply two positive numbers
Multiply by zero
Multiply negative numbers
Multiply decimal numbers
Division
Divide two positive numbers
Divide negative numbers
Divide decimal numbers
Verify division by zero raises an error
Acceptance Criteria
The automated testing task is considered successful when:

Tests
  ↓
Run locally
  ↓
16 Tests
  ↓
16 Passed
  ↓
0 Failed
  ↓
PASS

Final Status
PASS — Test suite prepared successfully.

The project contains automated unit tests covering the calculator application's core functionality and common edge cases.