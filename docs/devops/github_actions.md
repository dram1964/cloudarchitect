[GitHub Actions](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions) are the CI/CD platform for GitHub repositories, that allow you to trigger workflows 
in response to events in your repositories.

# GitHub Action Components
<dl>
  <dt>Workflows</dt>
  <dd>A workflow is a configurable automated process that will run one or more jobs. Workflows are defined in YAML files stored in the <code>.github/workflows</code> directory</dd>
  <dt>Events</dt>
  <dd>Events are specific activities in the repository that can trigger a workflow. These include: <ol><li>pull requests</li><li>opening an issue</li><li>commits</li></ol></dd>
  <dt>Jobs</dt>
  <dd>A Job is a set of steps in a workflow that execute on the same runner. Each step is either a <b>shell script</b> or an <b>action</b></dd>
  <dt>Actions</dt>
  <dd>An action is a custom application for GitHub Actions platform that performs a complex, repeatable task. The action can be defined in one of <ol><li>a public repository</li><li>the same repository as the workflow file</li><li>a published Docker container image</li></ol></dd>
  <dt>Runners</dt>
  <dd>A runner is a server used to run workflows. Each runner can run a single job at a time</dd>
</dl>

# Example Workflow Definition
Workflows are defined in YAML files

    name: my-github-workflow

    on: # define Events that will trigger the workflow
    push:
        branches: [main, develop, release]
    schedule:
        # 1am each night
        - cron: "0 1 * * *"

    jobs: 
    run-first-job:
        name: Run the first job
        runs-on: ubuntu:latest  
        if: github.ref == 'refs/heads/main'
        steps:
        - name: Checkout
            uses: actions/checkout@v3
        - name: Install Node
            uses: actions/setupnode@v3
            with:
            node-version: '14'
        - name: Run the Hello World Action
            uses: ./.github/actions/hello-world-action
            with:
            username: "John Doe"
            secrets:
            password: $&#123;&#123; secrets.JOHN_PASSWD &#125;&#125;
        - name: Salutation
            run: |
            echo "Thanks for running the workflow"
