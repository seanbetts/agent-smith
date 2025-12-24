# Test Coverage Analysis

Analysis of current test coverage and identification of gaps in the sideBar backend.

**Date**: 2025-12-23
**Total Test Files**: 10 (after removing list_skills tests)

---

## Current Test Coverage

### ✅ API Services (Well Tested)

| Component | Test File | Status | Notes |
|-----------|-----------|--------|-------|
| NotesService | `tests/api/test_notes_service.py` | ✅ Tested | CRUD operations for notes |
| WebsitesService | `tests/api/test_websites_service.py` | ✅ Tested | CRUD operations for websites |

### ✅ Security Layer (Well Tested)

| Component | Test File | Status | Notes |
|-----------|-----------|--------|-------|
| PathValidator | `tests/api/test_path_validator.py` | ✅ Tested | Workspace path validation |
| AuditLogger | `tests/api/test_audit_logger.py` | ✅ Tested | Security audit logging |

### ✅ Execution Layer (Tested)

| Component | Test File | Status | Notes |
|-----------|-----------|--------|-------|
| SkillExecutor | `tests/api/test_skill_executor.py` | ✅ Tested | Subprocess execution of skills |

### ✅ Authentication (Tested)

| Component | Test File | Status | Notes |
|-----------|-----------|--------|-------|
| Auth middleware | `tests/api/test_auth.py` | ✅ Tested | Bearer token authentication |

### ✅ MCP Integration (Tested)

| Component | Test File | Status | Notes |
|-----------|-----------|--------|-------|
| MCP Client | `tests/test_mcp_client.py` | ✅ Tested | MCP client implementation |
| MCP Integration | `tests/test_mcp_integration.py` | ✅ Tested | End-to-end MCP tests |

### ✅ Skills (Partial Coverage)

| Skill | Test File | Status | Notes |
|-------|-----------|--------|-------|
| skill-creator | `tests/skills/skill_creator/test_quick_validate.py` | ✅ Tested | Validation only |

### ✅ Utility Scripts (Partial Coverage)

| Script | Test File | Status | Notes |
|--------|-----------|--------|-------|
| add_skill_dependencies | `tests/scripts/test_add_skill_dependencies.py` | ✅ Tested | Dependency management |

---

## ❌ Missing Test Coverage

### Critical Components Without Tests

| Component | Location | Priority | Notes |
|-----------|----------|----------|-------|
| **ToolMapper** | `api/services/tool_mapper.py` | 🔴 HIGH | Maps MCP tools, executes tools - CRITICAL |
| **ClaudeClient** | `api/services/claude_client.py` | 🔴 HIGH | SSE streaming, tool execution orchestration |
| **Chat Router** | `api/routers/chat.py` | 🔴 HIGH | Main chat endpoint, SSE events |
| **Conversations Router** | `api/routers/conversations.py` | 🟡 MEDIUM | Conversation CRUD |
| **Files Router** | `api/routers/files.py` | 🟡 MEDIUM | Notes endpoints |
| **Websites Router** | `api/routers/websites.py` | 🟡 MEDIUM | Websites endpoints |
| **Scratchpad Router** | `api/routers/scratchpad.py` | 🟢 LOW | Scratchpad endpoints |
| **Health Router** | `api/routers/health.py` | 🟢 LOW | Health check endpoints |

### Database Layer

| Component | Location | Priority | Notes |
|-----------|----------|----------|-------|
| **Database Models** | `api/db/models/` | 🟡 MEDIUM | Note, Website, Conversation models |
| **Database Session** | `api/db/session.py` | 🟢 LOW | Session management |

### Configuration

| Component | Location | Priority | Notes |
|-----------|----------|----------|-------|
| **Settings** | `api/config.py` | 🟢 LOW | Environment configuration |

### Skills (Recently Refactored)

| Skill | Scripts | Priority | Notes |
|-------|---------|----------|-------|
| **notes** | `save_markdown.py`, `delete_note.py` | 🔴 HIGH | Database-backed, newly added |
| **web-save** | `save_url.py`, `delete_website.py` | 🔴 HIGH | Database-backed, newly added |
| **fs** | `list.py`, `read.py`, `write.py`, `search.py` | 🟡 MEDIUM | Core filesystem operations |

---

## Test Environment Issues

### Current Blockers

1. **Missing Environment Variables**
   - Tests fail because `bearer_token` and `anthropic_api_key` are required by Settings
   - Need to add fixtures to mock these in `conftest.py`

2. **Missing pytest-cov**
   - `pyproject.toml` references pytest-cov but it's not installed
   - Either install it or remove from test config

3. **Deleted Skills**
   - ~~`tests/skills/list_skills/`~~ - REMOVED (skill deleted)

---

## Recommended Test Priority

### Phase 1: Critical Path (HIGH Priority)

**Goal**: Test the core request flow from chat endpoint → tool execution → SSE response

1. **ToolMapper Tests** (`test_tool_mapper.py`)
   - Test `get_claude_tools()` returns correct tool definitions
   - Test `execute_tool()` for each tool type:
     - fs_list, fs_read, fs_write, fs_search
     - notes_create, notes_update, notes_delete
     - website_save, website_delete
   - Test error handling and validation
   - Test audit logging integration

2. **ClaudeClient Tests** (`test_claude_client.py`)
   - Test SSE event streaming
   - Test tool execution orchestration
   - Test SSE event emission (note_created, note_updated, etc.)
   - Test error handling
   - Mock Anthropic API responses

3. **Chat Router Tests** (`test_chat.py`)
   - Test POST /api/chat endpoint
   - Test SSE stream response
   - Test error handling
   - Integration test with mocked ClaudeClient

4. **Notes & Websites Script Tests**
   - Test `save_markdown.py` with `--database` flag
   - Test `delete_note.py` with `--database` flag
   - Test `save_url.py` with `--database` flag
   - Test `delete_website.py` with `--database` flag

### Phase 2: Router Coverage (MEDIUM Priority)

**Goal**: Test all API endpoints for correct behavior

5. **Conversations Router** (`test_conversations.py`)
   - Test list conversations
   - Test get conversation
   - Test delete conversation
   - Test update title

6. **Files Router** (`test_files.py`)
   - Test list notes
   - Test get note
   - Test update note
   - Test delete note

7. **Websites Router** (`test_websites.py`)
   - Test list websites
   - Test get website
   - Test delete website

### Phase 3: Database & Models (MEDIUM Priority)

**Goal**: Test database layer integrity

8. **Database Models Tests** (`test_models.py`)
   - Test Note model CRUD
   - Test Website model CRUD
   - Test Conversation model CRUD
   - Test relationships and constraints

### Phase 4: Skills Coverage (LOWER Priority)

**Goal**: Test individual skill scripts

9. **fs Skill Tests** (`test_fs_skill.py`)
   - Test list.py
   - Test read.py
   - Test write.py
   - Test search.py

10. **Other Skills** (as needed)
    - skill-creator (already has some tests)
    - Other high-value skills when exposed

---

## Test Infrastructure Improvements Needed

### 1. Environment Setup

Add to `conftest.py`:
```python
@pytest.fixture(autouse=True)
def mock_settings(monkeypatch):
    """Mock required environment variables for all tests."""
    monkeypatch.setenv("BEARER_TOKEN", "test-bearer-token-123")
    monkeypatch.setenv("ANTHROPIC_API_KEY", "test-anthropic-key-123")
    monkeypatch.setenv("DATABASE_URL", "sqlite:///:memory:")  # In-memory test DB
```

### 2. Database Fixtures

Add database fixtures for service tests:
```python
@pytest.fixture
def test_db():
    """Create a test database session."""
    from sqlalchemy import create_engine
    from sqlalchemy.orm import sessionmaker
    from api.db.base import Base

    engine = create_engine("sqlite:///:memory:")
    Base.metadata.create_all(engine)
    TestSessionLocal = sessionmaker(bind=engine)

    db = TestSessionLocal()
    try:
        yield db
    finally:
        db.close()
```

### 3. API Test Client

Add FastAPI test client fixture:
```python
@pytest.fixture
def test_client():
    """Create a test client for API endpoints."""
    from fastapi.testclient import TestClient
    from api.main import app

    return TestClient(app)
```

### 4. Coverage Configuration

Either:
- Install `pytest-cov`: Add to dev dependencies
- Or remove coverage from `pyproject.toml` if not needed yet

---

## Success Metrics

### Phase 1 Complete When:
- [ ] ToolMapper has 80%+ coverage
- [ ] ClaudeClient has 80%+ coverage
- [ ] Chat router has 80%+ coverage
- [ ] Database-backed skill scripts have 80%+ coverage
- [ ] All critical path tests pass

### Phase 2 Complete When:
- [ ] All routers have 80%+ coverage
- [ ] All router tests pass

### Phase 3 Complete When:
- [ ] Database models have 90%+ coverage
- [ ] All model tests pass

### Overall Goal:
- [ ] 80%+ overall code coverage
- [ ] All critical paths tested
- [ ] CI/CD can run tests automatically
- [ ] Tests run in < 30 seconds

---

## Next Steps

1. **Fix test environment** - Add environment variable mocking to conftest.py
2. **Fix pytest config** - Install pytest-cov or remove from config
3. **Write ToolMapper tests** - Highest priority, core functionality
4. **Write ClaudeClient tests** - Second priority, SSE streaming
5. **Continue with Phase 1** - Complete critical path coverage
