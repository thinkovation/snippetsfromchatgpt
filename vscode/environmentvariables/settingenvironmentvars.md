# Setting environment variables for a project

## The prompt:
I am using ms visual code - for a particular project I would like to set certain environment variables - specifically set "VCPKG_ROOT=C:\path\to\vcpkg"
set PATH=%VCPKG_ROOT%;%PATH% . I want this to happen whenever I open this project.

## The Response


To ensure that specific environment variables are set whenever you open a particular project in Visual Studio Code, you can use the `.vscode` directory within your project to store configuration files. Here are the steps to achieve this:

1. **Create a `.vscode` Directory**:
   - In the root of your project directory, create a folder named `.vscode` if it doesn't already exist.

2. **Create or Edit `settings.json`**:
   - Inside the `.vscode` directory, create a file named `settings.json` if it doesn't already exist.
   - Add the following configuration to set the `VCPKG_ROOT` environment variable and modify the `PATH`:

   ```json
   {
       "terminal.integrated.env.windows": {
           "VCPKG_ROOT": "C:\\path\\to\\vcpkg",
           "PATH": "${env:VCPKG_ROOT};${env:PATH}"
       }
   }
   ```

3. **Create a Tasks Configuration** (Optional):
   - You can also create a `tasks.json` file in the `.vscode` directory to define tasks that use these environment variables. Here's an example:

   ```json
   {
       "version": "2.0.0",
       "tasks": [
           {
               "label": "Run your task",
               "type": "shell",
               "command": "echo %VCPKG_ROOT%",
               "problemMatcher": [],
               "options": {
                   "env": {
                       "VCPKG_ROOT": "C:\\path\\to\\vcpkg",
                       "PATH": "${env:VCPKG_ROOT};${env:PATH}"
                   }
               }
           }
       ]
   }
   ```

4. **Reload VS Code**:
   - After saving the configuration files, reload VS Code to apply the changes. The specified environment variables will be set whenever you open a terminal within this project.

### Alternative: Using `.env` File
Visual Studio Code also supports defining environment variables in a `.env` file which can be useful for larger projects. Here’s how you can set this up:

1. **Create a `.env` File**:
   - In the root of your project, create a file named `.env`.
   - Add the following lines to set the `VCPKG_ROOT` environment variable and update the `PATH`:

   ```env
   VCPKG_ROOT=C:\path\to\vcpkg
   PATH=${VCPKG_ROOT};${PATH}
   ```

2. **Configure VS Code to Load the `.env` File**:
   - Open the `settings.json` file inside the `.vscode` directory and add the following configuration:

   ```json
   {
       "python.envFile": "${workspaceFolder}/.env"
   }
   ```

   Note: While this setting is specific to Python projects, it can be adapted to other languages by using similar configurations in corresponding extensions.

By following these steps, you ensure that your specified environment variables are set up automatically whenever you open your project in Visual Studio Code.