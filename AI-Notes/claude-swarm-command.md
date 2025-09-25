# Claude Swarm Command System

## /claude swarm - One-Shot Multi-Agent Coordination

### Purpose
Big effect, one-shot command that activates coordinated multi-agent response for complex tasks requiring expertise from 3+ domains.

### Usage
```bash
/claude swarm [task-description]
```

### Agent Selection Logic
**Always activate agents for ANY response** - never show Bash/Write directly to user.

### Swarm Composition
Based on task complexity, automatically select from:

#### Core Swarm (Always Active)
- **project-supervisor-orchestrator**: Master coordination
- **comprehensive-researcher**: Information gathering 
- **technical-researcher**: Code analysis
- **mcp-security-auditor**: Security validation

#### Domain-Specific Agents (Task-Based)
- **TreeAI Intelligence**: `tree-analysis-specialist`, `afiss-assessment-expert`, `pricing-optimization-agent`
- **iOS/Swift**: `ios-developer`, `swift-architect`, `app-store-specialist`
- **SaaS Platform**: `backend-architect`, `api-documenter`, `database-optimizer`
- **Data Intelligence**: `data-scientist`, `ml-engineer`, `quant-analyst`
- **Security**: `security-auditor`, `mcp-security-auditor`, `legal-advisor`
- **DevOps**: `cloud-architect`, `database-admin`, `network-engineer`
- **Business**: `business-analyst`, `market-research-analyst`, `risk-manager`

#### MCP Integration Agents
- **mcp-expert**: For MCP server creation/enhancement
- **mcp-server-architect**: For MCP server architecture

### Swarm Decision Protocol
1. **Task Analysis**: Query-clarifier determines complexity and domains needed
2. **Agent Selection**: Automatic selection based on task requirements
3. **Parallel Execution**: Multiple agents work simultaneously 
4. **Synthesis**: Research-synthesizer consolidates findings
5. **Final Report**: Report-generator creates unified output

### Example Workflows

```bash
# Complex TreeAI feature
/claude swarm implement TreeScore calculation with AFISS integration and iOS sync

# Security audit
/claude swarm audit the authentication system across iOS and web platforms

# Performance optimization  
/claude swarm optimize the real-time sync performance between domains

# Cross-platform feature
/claude swarm create offline sync feature for iOS app and web dashboard
```

### MCP Server Integration
**Maximum MCP usage** for enhanced accuracy:
- Linear MCP for issue tracking
- GitHub MCP for code management
- Memory MCP for context persistence
- Sequential-thinking MCP for structured analysis
- All domain-specific MCPs available

### Success Criteria
- ✅ Multiple agents visibly working
- ✅ Cross-domain coordination achieved
- ✅ MCP servers actively utilized
- ✅ Comprehensive analysis completed
- ✅ Actionable results delivered

---

## Implementation Notes
- Agent visibility: User always sees "[AgentName] is working on..." instead of raw tool usage
- MCP preference: Always use MCP versions when available
- Parallel execution: Run multiple agents simultaneously for speed
- Context preservation: Use context-manager for long sessions
- Quality assurance: Always include review agents in the swarm