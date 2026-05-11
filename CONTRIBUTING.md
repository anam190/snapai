# Contributing to SnapAI

Thank you for your interest in contributing to SnapAI! This document provides guidelines and instructions for contributing.

## Code of Conduct

- Be respectful and inclusive
- Focus on constructive feedback
- Help others learn and grow
- Report inappropriate behavior to maintainers

## Getting Started

### 1. Fork the Repository

```bash
git clone https://github.com/YOUR_USERNAME/snapai.git
cd snapai
git remote add upstream https://github.com/anam190/snapai.git
```

### 2. Create a Branch

```bash
git checkout -b feature/your-feature-name
# or
git checkout -b fix/your-bug-fix
```

Branch naming conventions:
- `feature/` - New features
- `fix/` - Bug fixes
- `docs/` - Documentation
- `test/` - Tests
- `refactor/` - Code improvements

### 3. Set Up Development Environment

```bash
# Backend
cd backend
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Frontend
cd ../frontend
npm install
```

## Development Workflow

### Backend Development

```bash
cd backend

# Run with auto-reload
uvicorn app:app --reload

# Run tests
pytest

# Format code
black .

# Lint code
flake8

# Type check
mypy .
```

### Frontend Development

```bash
cd frontend

# Start dev server
npm run dev

# Run tests
npm run test

# Format code
npm run format

# Lint code
npm run lint

# Build production
npm run build
```

## Code Style

### Python (Backend)

We follow PEP 8 with these tools:

```bash
# Format with Black
black backend/

# Lint with flake8
flake8 backend/

# Type check with mypy
mypy backend/
```

**Key Rules:**
- Max line length: 88 characters
- Use type hints on all functions
- Docstrings for all modules, classes, functions
- 4-space indentation

### TypeScript/JavaScript (Frontend)

```bash
# Format with Prettier
npx prettier --write .

# Lint with ESLint
npm run lint
```

**Key Rules:**
- Semicolons required
- Single quotes preferred
- 2-space indentation
- Type annotations required

### Commit Messages

```
Type: Subject

Body (optional)

Footer (optional)
```

Types:
- `feat:` - New feature
- `fix:` - Bug fix
- `docs:` - Documentation
- `style:` - Formatting
- `refactor:` - Code improvement
- `perf:` - Performance
- `test:` - Tests
- `chore:` - Build/dependencies

Examples:
```
feat: add sentiment analysis to messages

fix: resolve WebSocket connection timeout

docs: update setup guide for MacOS
```

## Pull Request Process

### 1. Before Submitting

- [ ] Code follows style guidelines
- [ ] All tests pass: `pytest` (backend) + `npm run test` (frontend)
- [ ] Added tests for new functionality
- [ ] Updated documentation if needed
- [ ] No merge conflicts
- [ ] Commits are squashed/organized

### 2. Create Pull Request

Fill out the PR template completely:

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
How to test these changes

## Screenshots
If UI changes, add screenshots

## Checklist
- [ ] Tests pass
- [ ] Code formatted
- [ ] Documentation updated
```

### 3. Review Process

- Maintainers will review your PR
- Address feedback and make requested changes
- Re-request review after updates
- PR merges after approval

## Testing Requirements

### Backend Tests

```bash
cd backend

# Run all tests
pytest

# Run specific test file
pytest tests/test_auth.py

# Run with coverage
pytest --cov=. --cov-report=html

# Run with verbose output
pytest -v
```

**Test Coverage Requirements:**
- Minimum 80% coverage
- All new functions must have tests
- Critical paths must be tested

### Frontend Tests

```bash
cd frontend

# Run all tests
npm run test

# Watch mode
npm run test:watch

# With coverage
npm run test:coverage
```

## Documentation

### Backend Documentation

Use docstrings for all public functions:

```python
def send_message(conversation_id: str, message: str) -> Dict:
    """
    Send a message in a conversation.
    
    Args:
        conversation_id: UUID of the conversation
        message: Message content
    
    Returns:
        Dictionary containing message ID and response
    
    Raises:
        NotFoundError: If conversation not found
        ValidationError: If message is empty
    """
    pass
```

### Frontend Documentation

Use JSDoc for components:

```typescript
/**
 * Chat message component
 * 
 * @param {Message} message - Message object
 * @param {boolean} isCurrentUser - Whether message from current user
 * @returns {JSX.Element}
 */
const MessageBubble = ({ message, isCurrentUser }: Props) => {
  return <div>...</div>;
};
```

## Adding Features

### New API Endpoint

1. **Create schema** (`backend/schemas.py`):
```python
class MyRequestSchema(BaseModel):
    field1: str
    field2: int
```

2. **Create model** if needed (`backend/models.py`):
```python
class MyModel(Base):
    __tablename__ = "my_table"
    id: Mapped[uuid.UUID]
    field1: Mapped[str]
```

3. **Create route** (`backend/routes/my_route.py`):
```python
@router.post("/endpoint")
async def my_endpoint(
    data: MyRequestSchema,
    current_user: User = Depends(get_current_user)
):
    # Implementation
    return {"result": "success"}
```

4. **Add tests** (`backend/tests/test_my_route.py`):
```python
def test_my_endpoint():
    response = client.post("/api/my/endpoint", json={...})
    assert response.status_code == 200
```

### New Frontend Component

1. **Create component** (`frontend/src/components/MyComponent.tsx`):
```typescript
interface Props {
  prop1: string;
}

const MyComponent: React.FC<Props> = ({ prop1 }) => {
  return <div>{prop1}</div>;
};

export default MyComponent;
```

2. **Create tests** (`frontend/src/components/__tests__/MyComponent.test.tsx`):
```typescript
describe("MyComponent", () => {
  it("renders correctly", () => {
    render(<MyComponent prop1="test" />);
    expect(screen.getByText("test")).toBeInTheDocument();
  });
});
```

3. **Use in app** (`frontend/src/App.tsx`):
```typescript
import MyComponent from "./components/MyComponent";
```

## Reporting Bugs

### Bug Report Template

```markdown
## Description
Clear description of the bug

## Steps to Reproduce
1. Step 1
2. Step 2
3. Step 3

## Expected Behavior
What should happen

## Actual Behavior
What actually happens

## Environment
- OS: [e.g., MacOS]
- Python: [e.g., 3.11]
- Node: [e.g., 18]
- Browser: [e.g., Chrome]

## Logs/Screenshots
Any relevant logs or screenshots
```

## Feature Requests

### Feature Request Template

```markdown
## Description
Clear description of the requested feature

## Problem It Solves
Why this feature is needed

## Proposed Solution
How to implement it

## Alternatives
Other possible approaches

## Additional Context
Any other information
```

## Performance Tips

### Backend Optimization

- Use database indexes for frequent queries
- Implement caching with Redis
- Use async/await properly
- Batch operations when possible
- Profile with `cProfile` for bottlenecks

### Frontend Optimization

- Use `React.memo` for expensive components
- Implement code splitting
- Lazy load images
- Minimize bundle size
- Use production builds for testing

## Security Considerations

### Before Submitting

- [ ] No hardcoded secrets/API keys
- [ ] No sensitive data in logs
- [ ] Input validation on all endpoints
- [ ] SQL injection prevention
- [ ] CORS headers configured properly
- [ ] Rate limiting considered

## Questions?

- Open a [GitHub Discussion](https://github.com/anam190/snapai/discussions)
- Check existing [Issues](https://github.com/anam190/snapai/issues)
- Review [Documentation](./README.md)

## Recognition

Contributors will be:
- Added to CONTRIBUTORS.md
- Mentioned in release notes
- Given credit in README

Thank you for contributing to SnapAI! 🎉
