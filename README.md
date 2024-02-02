# Heroku Buildpack for Multiple SSH Keys

This Heroku buildpack is adapted from [`heroku-buildpack-ssh-key`](https://github.com/heroku/heroku-buildpack-ssh-key) by Heroku. It is specifically designed to enable a Heroku application to access multiple private GitHub repositories, which is particularly useful for projects that inherit from several private repositories.

## Features

- Dynamically configures the SSH environment to use multiple SSH keys.
- Facilitates accessing multiple private GitHub repositories during the build process.
- Tailored for projects with dependencies on private repositories.

## Buildpack Order

This buildpack must be the first in the sequence of buildpacks used by your Heroku application. It sets up the SSH keys before any other buildpack attempts to access private repositories. You can manage and reorder your buildpacks in the Heroku dashboard or using the Heroku CLI.

Refer to [Heroku: Using multiple buildpacks for an app](https://devcenter.heroku.com/articles/using-multiple-buildpacks-for-an-app) for more details.

## Usage

1. **Setting Up SSH Keys:**
   Generate SSH keys for each private repository you need to access. Add each public key (`.pub` file) to its respective GitHub repository as a deploy key.

2. **Configuring Heroku:**
   For each private repository, you need to create a corresponding configuration variable (config var) in your Heroku app. It's crucial that each config var begins with `BSK_`. Follow the naming format: `BSK_REPONAME_UPPERCASE`, where `REPONAME_UPPERCASE` represents the name of the repository in uppercase letters. Assign the private SSH key for the respective repository as the value of this config var.


3. **Adding the Buildpack:**
   Add this buildpack to your Heroku app. Ensure it's first in the list of buildpacks:

   ```bash
   heroku buildpacks:add --index 1 https://github.com/CodigoSemilla/heroku-buildpack-for-multiple-ssh-keys.git -a <app-name>
   ```

   This command ensures this buildpack is the first one Heroku uses during the build process.

4. **Deploying Your Application:**
   Upon deployment, the buildpack sets up the SSH keys. These keys will be available for accessing private repositories required by your application.

## Tutorial

### Generating SSH Keys

For each private repository:

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
# When prompted, give a unique name to each key
```

### Adding Deploy Keys to GitHub Repositories

For repositories like `sample_private_repo_01`:

- Go to the repository on GitHub.
- Navigate to `Settings` > `Deploy keys`.
- Click `Add deploy key`, paste the public key, and save.

### Setting Config Vars in Heroku

For each SSH key, set a config var in your Heroku app:

- Go to your app's dashboard on Heroku.
- Navigate to `Settings` > `Config Vars`.
- Add a new config var with the key as `BSK_REPONAME_UPPERCASE` and paste the entire private key as the value.

### Add and Deploy this Buildpack

- Add this buildpack to your Heroku app.
- Deploy your application as usual. The buildpack will configure the SSH environment with the keys.

## Note

- Ensure that the SSH keys are kept secure and have minimum necessary access rights.
- For specific repository access, configure your build process to use the correct SSH key for each repository.

## Example Usage in a Ruby on Rails App

This buildpack is especially useful for Ruby on Rails applications that depend on private gems hosted in separate GitHub repositories. Below is a guide on how to set up your Rails application to use this buildpack effectively.

### Configuring the Gemfile

In your Rails application's `Gemfile`, specify your private gems by pointing to their GitHub repositories. Use the modified GitHub repository URL format to match the SSH configuration set up by the buildpack. For example:

```ruby
gem "secret_repo_01",  git: "git@github.com_secret_repo_01:username/secret_repo_01.git"
gem "secret_repo_02",  git: "git@github.com_secret_repo_02:username/secret_repo_02.git"
gem "secret_repo_03",  git: "git@github.com_secret_repo_03:username/secret_repo_03.git"
```

### SSH Configuration

Ensure your local SSH configuration (`~/.ssh/config`) includes entries for each private repository. This setup allows Git to recognize which SSH key to use for each repository:

```
Host github.com_secret_repo_01
    HostName github.com
    IdentityFile ~/.ssh/id_rsa_secret_repo_01

Host github.com_secret_repo_02
    HostName github.com
    IdentityFile ~/.ssh/id_rsa_secret_repo_02

Host github.com_secret_repo_03
    HostName github.com
    IdentityFile ~/.ssh/id_rsa_secret_repo_03
```

### Handling Dependency Caching on Heroku

Heroku caches dependencies to speed up subsequent builds. However, if you encounter issues with private gems not updating, you may need to clear the build cache. To do this, use the Heroku CLI:

1. Install the Heroku Repo plugin:
   ```bash
   heroku plugins:install heroku-repo
   ```

2. Clear the build cache:
   ```bash
   heroku repo:purge_cache -a app-name
   ```

3. Redeploy your application.

This process forces Heroku to fetch fresh copies of all dependencies, including any updates to your private gems.

## Disclaimer

This buildpack is independently developed and is not endorsed, certified, or reviewed by Heroku. It is based on the original `heroku-buildpack-ssh-key` from Heroku and adapted for specific use cases. Users should exercise discretion and evaluate the suitability of this buildpack for their projects.

## Acknowledgements

This buildpack is adapted from `heroku-buildpack-ssh-key` by Heroku. The original buildpack is designed to set up a single SSH key for accessing private repositories. This buildpack extends that functionality to support multiple SSH keys, making it easier to access multiple private repositories during the build process.
