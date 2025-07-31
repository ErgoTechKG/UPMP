# Unified Project Structure

```plaintext
srmp/
├── .github/                    # CI/CD workflows
│   └── workflows/
│       ├── ci.yaml            # Test and lint
│       └── deploy.yaml        # Deploy to Alibaba Cloud
├── apps/                       # Application packages
│   ├── web/                    # Frontend application
│   │   ├── src/
│   │   │   ├── components/     # UI components
│   │   │   ├── pages/          # Page components/routes
│   │   │   ├── hooks/          # Custom React hooks
│   │   │   ├── services/       # API client services
│   │   │   ├── stores/         # Zustand stores
│   │   │   ├── styles/         # Global styles/themes
│   │   │   ├── utils/          # Frontend utilities
│   │   │   ├── i18n/           # Translations
│   │   │   └── main.tsx        # Entry point
│   │   ├── public/             # Static assets
│   │   ├── tests/              # Frontend tests
│   │   ├── vite.config.ts      # Vite configuration
│   │   └── package.json
│   └── api/                    # Backend application
│       ├── src/
│       │   ├── modules/         # NestJS modules
│       │   │   ├── auth/
│       │   │   ├── users/
│       │   │   ├── projects/
│       │   │   ├── matching/
│       │   │   ├── training/
│       │   │   └── notifications/
│       │   ├── common/          # Shared module resources
│       │   │   ├── decorators/
│       │   │   ├── filters/
│       │   │   ├── guards/
│       │   │   ├── interceptors/
│       │   │   └── pipes/
│       │   ├── config/          # Configuration
│       │   ├── database/        # Database module
│       │   └── main.ts          # Entry point
│       ├── tests/              # Backend tests
│       └── package.json
├── packages/                   # Shared packages
│   ├── shared/                 # Shared types/utilities
│   │   ├── src/
│   │   │   ├── types/          # TypeScript interfaces
│   │   │   ├── constants/      # Shared constants
│   │   │   ├── dto/            # Data transfer objects
│   │   │   └── utils/          # Shared utilities
│   │   └── package.json
│   ├── ui/                     # Shared UI components
│   │   ├── src/
│   │   │   ├── components/
│   │   │   └── styles/
│   │   ├── package.json
│   │   └── tsconfig.json
│   └── config/                 # Shared configuration
│       ├── eslint/
│       ├── typescript/
│       └── jest/
├── infrastructure/             # IaC definitions
│   ├── terraform/
│   │   ├── environments/
│   │   │   ├── dev/
│   │   │   ├── staging/
│   │   │   └── production/
│   │   ├── modules/
│   │   │   ├── ecs/
│   │   │   ├── rds/
│   │   │   ├── redis/
│   │   │   └── cdn/
│   │   └── main.tf
│   └── k8s/                    # Kubernetes manifests
│       ├── base/
│       └── overlays/
├── scripts/                    # Build/deploy scripts
│   ├── setup.sh               # Initial setup
│   ├── build.sh               # Build all packages
│   └── deploy.sh              # Deployment script
├── docs/                       # Documentation
│   ├── prd.md
│   ├── front-end-spec.md
│   ├── architecture.md
│   └── api/                    # API documentation
├── .env.example                # Environment template
├── nx.json                     # Nx configuration
├── package.json                # Root package.json
├── pnpm-workspace.yaml         # PNPM workspace config
└── README.md
```
