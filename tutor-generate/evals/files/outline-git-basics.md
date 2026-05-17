# Learning Outline: Git Basics

## Learner Profile
- **Current level**: Knows how to write code but never used version control. Comfortable with terminal/command line.
- **Depth-breadth strategy**: Breadth-first. Build a conceptual framework first. Each chapter teaches one core operation.

---

## Chapter 1: Why Version Control — Saving and Tracking Your Work
- **Prerequisites**: None
- **Teaching approach**:
  1. **Concrete example**: Imagine writing an essay — you save copies as essay-v1.doc, essay-v2.doc, essay-final.doc, essay-final-FINAL.doc. Show the chaos.
  2. **Pattern recognition**: Same pattern in code (backup folders), graphic design (layers/history), collaborative writing (track changes).
  3. **Abstract definition**: Version control = a system that records changes over time, lets you go back to any version, and shows who changed what.
- **Core takeaway**: Git is like a time machine for your code — you can save checkpoints and rewind anytime.
- **Omitted for now**: Git internals (objects, hashes), branching, remote repositories. [(有更多细节，后续展开)]
- **Transition to next**: Now that we know WHY we need version control, let's learn the first practical operation: saving our work.

## Chapter 2: Your First Git Workflow — init, add, commit
- **Prerequisites**: Chapter 1 (why version control)
- **Teaching approach**:
  1. **Concrete example**: A developer starts a new project. They create a file, then `git init` (start tracking), `git add` (stage the file), `git commit` (save the checkpoint). Show the full terminal session.
  2. **Pattern recognition**: Analogy — writing a report: draft (working directory) → proofread selection (staging area) → final save (commit). Photography: taking photos (working) → selecting shots (staging) → album (commit).
  3. **Abstract definition**: Working directory = where you edit files. Staging area (index) = what you intend to commit. Repository = the saved history of commits.
- **Core takeaway**: `git init` starts tracking, `git add` stages changes, `git commit` saves a checkpoint with a message.
- **Omitted for now**: `.gitignore`, `git status` details, `git diff`, reverting commits. [(有更多细节，后续展开)]
- **Transition to next**: Now you can save your own work. Next step: collaborating with others.
