# Peribolos configuration

Emporous uses Operate First's [peribolos-as-a-service](https://github.com/operate-first/peribolos-as-a-service) to create/update admins, teams, and repositories in GitHub.

## How to use

1. Verify peribolos configuration change with provided tools (Tooling is a WIP, follow the GitHub issue [here](https://github.com/emporous/.github/issues/26))
2. Open a pull request to add/change a member, admin, team,  or repository in the `.github` [repo](https://github.com/emporous/.github) on the `peribolos.yaml` file.
3. After the pull request is reviewed and merged, the changes will be applied by peribolos.

## Add a new member

To add a new user to be part of the GitHub Org you should add a new entry following the example

```yaml
orgs:
  emporous:
    ...
    members:
      - "username"
```

## Add a new team

```yaml
orgs:
  emporous:
    ...
    teams:
      newteam:
        description: "a new team"
        privacy: closed
        members: []
        repos:
          newrepo: write
```

### Added a member to a team

```yaml
orgs:
  emporous:
    ...
    teams:
      newteam:
        description: "a new team"
        privacy: closed
        members: 
          - "username"
        repos:
          newrepo: write
```

## Add a new repository or update existing ones

```yaml
orgs:
  emporous:
    ...
    repos:
      newrepo:
        default_branch: main
        description: "newrepo"
        has_projects: true
```