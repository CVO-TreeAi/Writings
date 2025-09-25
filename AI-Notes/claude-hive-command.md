# Claude Hive Command System

## /claude hive - Autonomous Self-Managed Workflows

### Purpose
Autonomous, self-managing agent system that runs on autopilot using Linear MCP for task management. Only stops to confirm critical decisions or get questions answered.

### Usage
```bash
/claude hive [initial-objective]
```

### Hive Autopilot Behavior
1. **Linear Integration**: Creates Linear issues for all sub-tasks
2. **Self-Management**: Agents autonomously break down work and execute
3. **Minimal Interruption**: Only asks user for:
   - Critical architectural decisions
   - Clarification on ambiguous requirements  
   - Permission for destructive operations
   - Final approval before major deployments
4. **Continuous Progress**: Always shows active agent work
5. **Maximum MCP Usage**: Leverages all available MCP servers

### Hive Architecture

#### Master Orchestrator
- **context-manager**: Manages long-running workflows
- **project-supervisor-orchestrator**: Coordinates agent sequences
- **research-orchestrator**: Manages research workflows

#### Autonomous Agent Pool
All agents available with automatic selection based on:
- Task requirements
- Domain expertise needed
- MCP server compatibility
- Workload distribution

#### Linear MCP Autopilot Integration
```typescript
interface HiveLinearIntegration {
  // Auto-create Linear issues for tasks
  createTaskIssue(task: TaskSpec): Promise<LinearIssue>;
  
  // Track progress automatically
  updateTaskProgress(issueId: string, progress: Progress): Promise<void>;
  
  // Auto-assign to appropriate agents
  assignToAgent(issueId: string, agent: AgentType): Promise<void>;
  
  // Close completed tasks
  completeTask(issueId: string, results: TaskResults): Promise<void>;
}
```

### Hive Decision Framework

#### Autonomous Decisions (No User Input)
- Agent selection and allocation
- Task breakdown and prioritization
- Code quality improvements
- Performance optimizations
- Documentation updates
- Test creation and execution
- Routine security checks

#### User Confirmation Required
- Destructive operations (file deletion, schema changes)
- Major architectural changes
- External API integrations
- Production deployments
- Security policy changes
- Breaking API changes

### MCP Server Orchestration

#### Required MCPs
- **Linear MCP**: Task management and tracking
- **GitHub MCP**: Code management and CI/CD
- **Memory MCP**: Context and learning persistence
- **Sequential-thinking MCP**: Structured problem solving

#### Domain MCPs (Auto-activated)
- **Cloudflare MCPs**: For web infrastructure
- **Database MCPs**: For data operations  
- **Container MCPs**: For deployment
- **Browser MCPs**: For testing and automation

### Hive Workflow Example

```bash
/claude hive create a complete user authentication system with iOS and web support

# Hive Response:
"🧠 Hive Intelligence Activated
📋 Creating Linear project: Authentication System Implementation
🤖 Agents deployed: security-auditor, ios-developer, backend-architect, mcp-expert

Agent Activity:
• [security-auditor] Designing OAuth 2.1 + PKCE security architecture
• [ios-developer] Planning SwiftUI authentication flows  
• [backend-architect] Architecting Convex auth functions
• [mcp-expert] Setting up authentication MCP servers

📊 Linear Issues Created:
- AUTH-1: Security architecture design
- AUTH-2: iOS authentication UI
- AUTH-3: Backend auth functions
- AUTH-4: MCP server integration
- AUTH-5: Cross-platform testing

🔄 Autopilot Mode: Agents will continue working autonomously. 
   Will only interrupt for critical decisions or clarifications."
```

### Continuous Operations

#### Background Processing
- Agents continue working between user sessions
- Progress persisted in Linear and Memory MCP
- Context maintained across restarts
- Learning and optimization ongoing

#### Progress Reporting
```typescript
interface HiveProgress {
  activeAgents: AgentActivity[];
  completedTasks: LinearIssue[];
  nextMilestones: Milestone[];
  blockingQuestions?: ClarificationRequest[];
  estimatedCompletion: Date;
}
```

#### Auto-Recovery
- Failed tasks automatically reassigned
- Stuck agents replaced with alternatives
- Context recovery from Memory MCP
- Graceful degradation on MCP failures

### Hive Learning Integration
- Success patterns stored in Memory MCP
- Failed approaches documented and avoided
- Agent performance tracking and optimization
- User preference learning and adaptation

### Emergency Controls
```bash
/claude hive pause    # Pause all autonomous operations
/claude hive resume   # Resume autonomous operations  
/claude hive status   # Get current hive status
/claude hive reset    # Reset hive state (emergency)
```

---

## Implementation Architecture

### Agent Visibility Protocol
**Always show agent work instead of raw tools:**
```
❌ User sees: "Running bash command: git status"
✅ User sees: "[technical-researcher] Analyzing repository status..."

❌ User sees: "Writing file: auth.ts" 
✅ User sees: "[backend-architect] Implementing authentication module..."
```

### MCP-First Approach
**Priority order for operations:**
1. Use specific MCP server if available
2. Use general-purpose agent with MCP tools
3. Use Claude Code built-in tools (last resort)

### Hive Memory Integration
```python
class HiveMemorySystem:
    def store_workflow_pattern(self, pattern: WorkflowPattern):
        """Store successful workflow patterns for reuse"""
        
    def learn_from_completion(self, task: CompletedTask):
        """Learn from completed tasks for future optimization"""
        
    def retrieve_similar_patterns(self, task: TaskSpec) -> List[WorkflowPattern]:
        """Find similar successful patterns for current task"""
```

This creates a truly autonomous, self-managing system where agents are always visible and MCP servers maximize accuracy and reliability.