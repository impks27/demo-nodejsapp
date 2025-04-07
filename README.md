# demo-nodejsapp

When aiming to speed up Node.js builds, the choice between caching the `.npm` folder and `node_modules` depends on your use case and build environment. Here are the key considerations for each approach:

# Caching `node_modules` vs `.npm`

### Caching `node_modules`

**Pros:**
- **Faster subsequent builds:** Since the `node_modules` directory contains all installed dependencies, caching it can significantly speed up the build process because you avoid the need to reinstall packages.
- **Ideal for CI/CD:** In continuous integration and deployment setups, caching `node_modules` can lead to faster build times and quicker feedback loops.

**Cons:**
- **Platform dependencies:** If your build environment changes (e.g., different OS or Node.js version), the cached `node_modules` may become invalid or cause issues.
- **Large cache size:** `node_modules` can become quite large, consuming significant storage space and potentially increasing the time required to store and retrieve the cache.

### Caching `.npm`

**Pros:**
- **Consistency:** The `.npm` cache contains downloaded packages, which can be reused across different environments and Node.js versions, leading to more consistent behavior.
- **Smaller cache size:** Typically, the `.npm` cache is smaller than `node_modules`, leading to faster cache storage and retrieval.

**Cons:**
- **Requires reinstallation:** Caching `.npm` still requires running `npm install`, which means dependencies will be installed again, although this process will be faster since packages are fetched from the cache rather than downloaded anew.
- **Dependency updates:** If dependencies are frequently updated, the benefits of caching `.npm` diminish, as you will still need to resolve and install new versions.

### Recommended Approach

For many CI/CD setups, a hybrid approach can be beneficial:

1. **Cache `.npm`:** This helps to speed up the dependency resolution and downloading process. 
2. **Use lock files (e.g., `package-lock.json`):** Ensure consistency of installed dependencies across environments.
3. **Selective `node_modules` caching (optional):** If you have a stable build environment and large dependencies that seldom change, selectively caching parts of `node_modules` can further speed up builds.

### Example CI/CD Configuration (e.g., GitHub Actions)

Here’s an example of how you might configure GitHub Actions to cache both `.npm` and `node_modules`:

```yaml
name: Node.js CI

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2

    - name: Cache npm modules
      uses: actions/cache@v2
      with:
        path: ~/.npm
        key: ${{ runner.os }}-npm-cache-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-npm-cache-

    - name: Cache node_modules
      uses: actions/cache@v2
      with:
        path: node_modules
        key: ${{ runner.os }}-node_modules-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-node_modules-

    - name: Install dependencies
      run: npm ci

    - name: Run build
      run: npm run build
```

In this example:
- The `.npm` cache speeds up the dependency resolution and download process.
- The `node_modules` cache speeds up builds if dependencies have not changed.

Choose the approach that best suits your project’s needs and build environment constraints.

# Caching for multi-module nodejs project

For a multi-module Node.js project managed by npm, you can cache and restore the `node_modules` directories to speed up the CI/CD process. Here's how you can achieve this with GitHub Actions:

### Example Using GitHub Actions

Let's assume you have a monorepo with the following structure:
```
/monorepo
├── package.json
├── package-lock.json
├── packages
    ├── module-a
    │   ├── package.json
    │   ├── package-lock.json
    ├── module-b
        ├── package.json
        ├── package-lock.json
```

### GitHub Actions Workflow

Create a workflow file `.github/workflows/ci.yml` with the following content:

```yaml
name: CI

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [14.x, 16.x]

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Cache root node_modules
      uses: actions/cache@v2
      with:
        path: node_modules
        key: ${{ runner.os }}-root-node_modules-${{ hashFiles('package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-root-node_modules-

    - name: Cache module-a node_modules
      uses: actions/cache@v2
      with:
        path: packages/module-a/node_modules
        key: ${{ runner.os }}-module-a-node_modules-${{ hashFiles('packages/module-a/package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-module-a-node_modules-

    - name: Cache module-b node_modules
      uses: actions/cache@v2
      with:
        path: packages/module-b/node_modules
        key: ${{ runner.os }}-module-b-node_modules-${{ hashFiles('packages/module-b/package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-module-b-node_modules-

    - name: Set up Node.js
      uses: actions/setup-node@v2
      with:
        node-version: ${{ matrix.node-version }}

    - name: Install root dependencies
      run: npm install

    - name: Install module-a dependencies
      working-directory: packages/module-a
      run: npm install

    - name: Install module-b dependencies
      working-directory: packages/module-b
      run: npm install

    - name: Build modules
      run: |
        cd packages/module-a && npm run build
        cd ../module-b && npm run build

    - name: Run tests
      run: npm test
```

### Key Points:

1. **Checkout Repository:**
   - `actions/checkout@v2` checks out your repository so the workflow can access its contents.

2. **Cache `node_modules` for Root and Modules:**
   - Use `actions/cache@v2` to cache the `node_modules` directory for the root and each module. The `key` and `restore-keys` help identify and restore the appropriate cache.

3. **Set Up Node.js:**
   - `actions/setup-node@v2` sets up the specified Node.js version.

4. **Install Dependencies:**
   - Run `npm install` in the root and each module directory to install dependencies. This will be faster if the cache is restored successfully.

5. **Build and Test:**
   - Navigate to each package directory and run the build and test commands.

### Notes:

- **Cache Keys:** The cache keys include the hash of the `package-lock.json` file to ensure that the cache is updated when dependencies change.
- **Cache Paths:** Adjust the paths to `node_modules` based on your project's structure.

This setup ensures that dependencies are cached and restored efficiently, speeding up the build and test processes for multi-module Node.js projects managed by npm.
- name: Update submodules (recursive, remote) and clean up credentials
  run: |
    # Inject GitHub App token for submodule access
    git config url."https://x-access-token:${{ steps.generate-token.outputs.token }}@github.com/".insteadOf "https://github.com/"

    # Update submodules to latest remote commits
    git submodule update --remote --recursive

    # Clean up the Git config
    git config --unset url."https://x-access-token:${{ steps.generate-token.outputs.token }}@github.com/".insteadOf