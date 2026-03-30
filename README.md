# Heroku Buildpack: Subdirectory with Monorepo Support

This buildpack allows you to deploy a subdirectory of your Git repository as a Heroku app, with **enhanced support for monorepo projects that share common dependencies**.

Add as the first buildpack in the chain. Set the `PROJECT_PATH` environment variable to point to your project root. It will be promoted to the slug's root along with any shared dependencies, then the following buildpack (e.g. Node.js) will finish slug compilation.

**Disclaimer:** This is a fork with monorepo enhancements. Always pin to a specific GitHub version. Provided as is.

## Features

- **Subdirectory deployment**: Deploy only a specific subdirectory of your repository
- **Monorepo support**: Automatically preserves shared dependencies (like a `common/` directory)
- **Clean structure**: Maintains relative paths so no code changes are needed
- **Wrapper package.json**: Creates a root-level package.json that delegates to your project

## How to use

1. **Clear existing buildpacks** (if necessary):
   ```bash
   heroku buildpacks:clear
   ```

2. **Add this buildpack** (replace with your fork URL):
   ```bash
   heroku buildpacks:add https://github.com/YOUR_USERNAME/subdir-heroku-buildpack
   ```

3. **Add your language buildpack**:
   ```bash
   heroku buildpacks:add heroku/nodejs
   ```
   Or whatever buildpack you need for your application.

4. **Configure PROJECT_PATH**:
   ```bash
   heroku config:set PROJECT_PATH=api
   ```
   Point to the subdirectory you want to deploy.

5. **Deploy**:
   ```bash
   git push heroku master
   ```

## How it works

### Standard Mode
If you only have a `PROJECT_PATH` configured, the buildpack:
1. Copies your subdirectory contents to a temporary location
2. Erases everything else
3. Moves the subdirectory contents to the project root
4. Normal Heroku slug compilation proceeds

### Monorepo Mode (with `common/` directory)
If your repository has both a `PROJECT_PATH` and a `common/` directory at the root, the buildpack:
1. Preserves the directory structure by keeping both directories
2. Creates a root `package.json` wrapper that:
   - Installs dependencies in your project directory
   - Installs extension/subdirectory dependencies if needed
   - Delegates `start` and `heroku-postbuild` commands to your project
3. Maintains all relative paths, so imports like `../../../common` continue to work

### Example monorepo structure

**Before buildpack:**
```
your-repo/
  api/
    package.json
    extensions/
      some-extension/
  common/          ← Shared types, utilities, etc.
  front/
  infra/
```

**After buildpack (BUILD_DIR):**
```
BUILD_DIR/
  package.json     ← Wrapper created by buildpack
  api/
    package.json   ← Your original
    extensions/
      some-extension/
  common/          ← Preserved!
```

## Configuration

### Required
- `PROJECT_PATH`: Path to your project subdirectory (e.g., `api`, `backend`, `server`)

### Optional
- Place a `common/` directory at the repository root to share code between projects
- The buildpack will automatically detect and preserve it

## Use Cases

Perfect for:
- Monorepo projects with shared TypeScript types/utilities
- Projects with multiple apps (frontend, backend, etc.) where you deploy each separately
- Clean separation of concerns while maintaining code reuse

## Original Buildpack

This is a fork of [timanovsky/subdir-heroku-buildpack](https://github.com/timanovsky/subdir-heroku-buildpack) with added monorepo support.
