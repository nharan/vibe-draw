# Claude Assistant Guidelines

## Commands
- Run API server: `python run.py`
- Run Celery worker: `celery -A worker worker --loglevel=info`
- Run with Docker: `docker-compose up`
- Install dependencies: `pip install -r requirements.txt`
- Run tests: `pytest tests/`
- Run single test: `pytest tests/test_file.py::test_function`
- Code formatting: `black .`
- Linting: `flake8 .`
- Type checking: `mypy app/`

## Code Style Guidelines
- **Imports**: Sort alphabetically; standard lib first, third-party second, local modules last
- **Formatting**: Follow PEP 8, line length 88 characters (Black defaults)
- **Type Hints**: Required for function parameters and return values
- **Naming**:
  - Classes: PascalCase
  - Functions/methods: snake_case
  - Variables: snake_case
  - Constants: UPPER_SNAKE_CASE
- **Documentation**: Docstrings for all public classes and functions (""" style)
- **Error Handling**: Use try/except with specific exception types, avoid bare except
- **Async**: Use asyncio for I/O-bound operations, especially with external APIs
- **Pydantic**: Use Pydantic models for data validation and serialization
- **Redis**: Prefer the singleton redis_service for all Redis interactions