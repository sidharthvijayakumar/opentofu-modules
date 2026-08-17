import pypandoc

content = r"""# Infrastructure Modules

A centralized repository for reusable infrastructure modules.

Each module is versioned independently. When a module change is merged into `master`, the Monorail release job identifies the changed module, determines the next version, and creates a Git tag for that module.

## Repository Structure

```text

.

├── modules/

│   ├── module-a/

│   │   ├── main.tf

│   │   ├── variables.tf

│   │   └── outputs.tf

│   │

│   ├── module-b/

│   │   ├── main.tf

│   │   ├── variables.tf

│   │   └── outputs.tf

│   │

│   └── ...

│

├── .gitlab-ci.yml

└── README.md

```

## Module Versioning

Modules use independent Git tags.

Example:

```text

module-a/v1.0.0

module-a/v1.1.0

module-a/v1.2.0

module-b/v1.0.0

module-b/v1.1.0

```

This allows each module to evolve independently without requiring a new version for every module in the repository.

## Release Flow

The expected release flow is:

```text

Developer

    |

    | Create branch

    v

Feature branch

    |

    | Merge Request

    v

master

    |

    v

GitLab CI/CD

    |

    v

Monorail Release Job

    |

    +--> Detect changed module

    |

    +--> Determine next version

    |

    +--> Create Git tag

    |

    v

module-a/v1.2.3

```

The tag points to the commit that was merged into `master`.

## Consuming a Module

Other repositories can consume a specific module version using its Git tag.

For example, a Terraform project can reference `module-a` version `1.2.3`:

```hcl

module "module_a" {

  source = "git::https://gitlab.example.com/platform/infrastructure-modules.git//modules/module-a?ref=module-a/v1.2.3"

  # Module inputs

}

```

Pinning to a tag ensures that the consuming project uses a known module version rather than whatever happens to be present on `master`.

## Example

Suppose the repository currently has:

```text

module-a/v1.0.0

module-a/v1.1.0

module-a/v1.2.0

```

A change is made to `module-a` and merged into `master`.

Monorail creates:

```text

module-a/v1.3.0

```

A consuming project can then upgrade explicitly:

```text

module-a/v1.2.0

        |

        v

module-a/v1.3.0

```

Other projects can remain on `v1.2.0` until they are ready to upgrade.

## Branching and Merge Strategy

### Feature branches

Create a feature branch for module changes:

```bash

git checkout -b feature/update-module-a

```

Make the required changes and push the branch.

### Merge Request

Create a Merge Request targeting:

```text

master

```

The module should be reviewed and validated before merging.

### After merge

Once the Merge Request is merged into `master`:

1. GitLab CI/CD starts the release pipeline.

2. Monorail identifies the changed module.

3. Monorail determines the next module version.

4. Monorail creates the Git tag.

5. The new tag becomes available to consuming repositories.

## Versioning Guidelines

Use semantic versioning:

```text

MAJOR.MINOR.PATCH

```

### MAJOR

Use for breaking changes.

Example:

```text

module-a/v1.4.2

        ↓

module-a/v2.0.0

```

### MINOR

Use for backward-compatible functionality.

Example:

```text

module-a/v1.4.2

        ↓

module-a/v1.5.0

```

### PATCH

Use for backward-compatible bug fixes.

Example:

```text

module-a/v1.4.2

        ↓

module-a/v1.4.3

```

The exact version-bump rules can be implemented by Monorail according to the organization's release conventions.

## CI/CD

The `.gitlab-ci.yml` is responsible for invoking the Monorail release process after changes are merged into `master`.

A simplified pipeline can look like:

```yaml

stages:

  - validate

  - test

  - release

validate:

  stage: validate

  script:

    - echo "Validate modules"

test:

  stage: test

  script:

    - echo "Run module tests"

monorail-release:

  stage: release

  rules:

    - if: '$CI_COMMIT_BRANCH == "master"'

  script:

    - echo "Invoke Monorail release process"

```

The actual Monorail implementation should be added according to the organization's Monorail tooling and authentication mechanism.

## Git Tags

Tags are created against the commit merged into `master`.

Example:

```text

Commit: abc1234

Module: module-a

Version: v1.2.3

Tag:

module-a/v1.2.3

        |

        +--> abc1234

```

Tags should be treated as immutable release references. Consumers should reference released tags instead of arbitrary commits whenever possible.

## Using Modules Across Projects

Multiple projects can consume different versions of the same module:

```text

Project A → module-a/v1.2.0

Project B → module-a/v1.2.3

Project C → module-a/v1.2.3

```

This allows teams to upgrade modules independently and reduces the risk of an unexpected module change affecting existing projects.

## Access Requirements

Consuming repositories need read access to this repository.

If the repository is private, the consuming CI/CD pipeline must authenticate to GitLab before downloading the module.

Authentication should use the organization's approved mechanism, such as:

- CI job tokens

- Deploy tokens

- Project/group access tokens

- Other approved GitLab authentication mechanisms

Secrets and access tokens must not be committed to this repository.

## Best Practices

- Keep modules independent and reusable.

- Version modules independently.

- Consume released tags rather than `master`.

- Treat release tags as immutable.

- Use Merge Requests for all changes.

- Validate and test modules before release.

- Avoid breaking changes in minor or patch releases.

- Document module inputs and outputs.

- Keep authentication credentials out of source code.

- Use protected tags where appropriate.

## Summary

This repository provides a centralized and versioned source for reusable infrastructure modules.

The intended workflow is:

```text

Change module

     |

     v

Create MR

     |

     v

Merge into master

     |

     v

Monorail

     |

     v

Create module-specific Git tag

     |

     v

Consume tagged version

from other project repositories

```

This approach provides independent module releases, reproducible deployments, and controlled upgrades across consuming projects.

