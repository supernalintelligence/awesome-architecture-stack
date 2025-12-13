# Contributing to Awesome Architecture Stack

Thank you for considering contributing! This guide helps the broader software community make informed architectural decisions.

## How to Contribute

### 1. Add a Tool or Service

When adding a new tool to a section:

**Required information:**
- **Name**: Official name
- **Category**: Which layer/section
- **Description**: One sentence (what problem it solves)
- **Pros**: 2-3 key strengths
- **Cons**: 2-3 limitations
- **Use cases**: When to use it

**Example:**
```markdown
| **Neon** | Serverless PostgreSQL | Auto-scaling, branching databases | Cold start latency, newer | Vercel projects, preview envs |
```

### 2. Update Trade-offs

Technology evolves. If a tool's pros/cons changed:
- Update the trade-off table
- Add a note: `(Updated 2025-01: X feature added)`
- Keep old info if historically relevant

### 3. Add Real-World Examples

Share lessons learned:
```markdown
**Lesson from Production**: We switched from MongoDB to PostgreSQL because...
```

Add under relevant section, marked as **Real-World Example**.

### 4. Improve Decision Frameworks

If you found a better way to evaluate choices:
- Add to "Decision Framework" sections
- Explain why your criteria matters
- Include trade-offs

## Guidelines

### ✅ DO:
- Be objective (present pros AND cons)
- Include trade-off tables (not just lists)
- Focus on "why" not just "what"
- Add examples from real projects
- Keep it concise (this is a reference, not a tutorial)

### ❌ DON'T:
- Promote your own product (unless genuinely awesome)
- Bash other tools (explain trade-offs objectively)
- Add niche tools (unless solving unique problem)
- Duplicate existing content
- Write marketing copy ("the best", "amazing", "revolutionary")

## Pull Request Process

1. **Fork the repo**
2. **Create a branch**: `git checkout -b add-neon-database`
3. **Make changes**: Follow formatting (see below)
4. **Test links**: Ensure all links work
5. **Submit PR**: Describe what you added and why

### Formatting

- **Tables**: Use Markdown tables for comparisons
- **Bold**: Use `**bold**` for emphasis
- **Code**: Use backticks for tool names: `Redis`
- **Sections**: Use `##` for main sections, `###` for subsections
- **Emoji**: Use sparingly (✅, ❌, 🚩 only)

## What We're Looking For

**High priority:**
- Missing architectural layers
- Better decision frameworks
- Real-world trade-off examples
- Updated tool information

**Low priority:**
- Minor wording changes
- Duplicate content
- Niche tools (unless unique use case)

## Code of Conduct

Be respectful:
- No inflammatory language
- No personal attacks
- No spam or self-promotion
- No plagiarism (cite sources)

## Questions?

Open an issue with the `question` label.

---

**Thank you for helping developers make better architecture decisions!**

