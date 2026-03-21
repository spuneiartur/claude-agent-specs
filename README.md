# Claude Agent Specs

Custom Claude Code agent specifications (CLAUDE.md files) and skills for our projects.

## Structure

```
express-js/    Backend project specs & skills (Express.js + MongoDB REST API)
next-js/       Frontend project specs & skills (Next.js + React + Tailwind)
```

Each directory contains:

- **CLAUDE.md** — Project-specific instructions for Claude Code (tech stack, coding standards, folder structure, commands)
- **skills/** — Claude agent skills that teach Claude repeatable development patterns

## Express.js Skills

| Skill | Description |
|-------|-------------|
| crud-resource-scaffolder | Scaffold full CRUD resources (controller, model, routes, schemas) |
| reference-field-manager | Manage reference/embedded fields in Mongoose schemas |
| public-endpoint-builder | Build public (unauthenticated) API endpoints |
| db-seed-generator | Generate database seed files |
| skill-creator | Create new skills from scratch |

## Next.js Skills

| Skill | Description |
|-------|-------------|
| page-scaffold | Scaffold new Next.js pages |
| component-factory | Generate reusable React components |
| form-builder | Build validated forms |
| admin-crud-scaffold | Scaffold admin CRUD interfaces |
| api-hook-wiring | Wire up API hooks with React Query |
| custom-hook | Generate custom React hooks |
| theme-factory | Apply and manage design themes |
| frontend-design | UI/UX design patterns |
| canvas-design | Canvas-based design generation |
| scroll-stop-builder | Build scroll-stop sections |
| 3d-website-asset-generation | Generate 3D website assets |
| web-artifacts-builder | Build standalone web artifacts |
| seo-strategy | SEO optimization patterns |
| website-speed-audit | Performance auditing |
| webapp-testing | Automated web app testing |
| brand-guidelines | Brand consistency enforcement |
| skill-creator | Create new skills from scratch |

## Usage

Copy the relevant `CLAUDE.md` and `skills/` directory into your project, or point Claude Code to this repo as a reference.
