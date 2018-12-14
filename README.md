# HolyLabRegistry
This is a HolyLab private registry to use lab packages according to julia 0.7 Pkg3 design.

# Usage
Clone this repository under DEPOT_PATH/registries. Then, we can use lab private
pakages as if they are registered ones. (Usually, DEPOT_PATH = /home/username/.julia)


# WIP
Still need to add most of our lab packages to this registry.


# To use git protocol in GitHub
(This instruction comes from https://help.github.com/articles/connecting-to-github-with-ssh/)

1. Generating a new SSH key at a local machine.
    - Open git bash and paste text below, substituting in your GitHub email address.
    ```
    $ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
    ```

    - When you're prompted to "Enter a file in which to save the key," press Enter. This accepts the default file location.
    ```
    Enter a file in which to save the key (/home/you/.ssh/id_rsa): [Press enter]
    ```

    - At the prompt, type a secure passphrase if you want.
    ```
    Enter passphrase (empty for no passphrase): [Type a passphrase]
    Enter same passphrase again: [Type passphrase again]
    ```

2. Adding your SSH key to the ssh-agent
    - Start the ssh-agent in the background.
    ```
    $ eval "$(ssh-agent -s)"
    Agent pid 59566
    ```

    - Add your SSH private key to the ssh-agent
    ```
    $ ssh-add ~/.ssh/id_rsa
    ```

3. Adding a new SSH key to your GitHub account
    - Copies the contents of the id_rsa.pub file in the local machine to your clipboard
    - Go to GitHub site
    - In the upper-right corner of any page, click your profile photo, then click Settings.
    - In the user settings sidebar, click SSH and GPG keys.
    - Click New SSH key or Add SSH key.
    - In the "Title" field, add a descriptive label for the new key.
    - Paste your copied pulic key into the "Key" field
    - Click Add SSH key.

# For package developer
- How to add your package to HolyLabRegistry
  - Make directory including Compat.toml, Deps.toml, Package.toml and Versions.toml under root directory (It would be easy to begin with just copy related files from other existing entry in the registry)
  - Edit those files appropriately.
    - Deps.toml : include all the dependencies
    - Package.toml : Package name, UUID, location of the repository
    - Versions.toml : git-tree-sha values according to version numbers.
      You can find this value with git command at the package root directory.
      ```
      $ git cat-file -p v0.1.0
      ```
      Then, You will see the value in the first line.
      If you just want to publish current 'master' branch, try this.
      ```
      $ git cat-file -p master
      ```
      In this case, every time master branch in the repository of the package is updated with new commit, you need to update this value also in this registry.
  - Add an entry of the package in Registry.toml file. As an example, add a below line if your package name is 'RFFT', directory name is also 'RFFT' and UUID is `3bd9afcd....`
    ```
    3bd9afcd-55df-531a-9b34-dc642dce7b95 = { name = "RFFT", path = "RFFT" }
    ```

- How to make a package be able to access this registry during the travis test
  - Include below lines in the script section of .travis.yml file in the root directory of your package (as an example, let your package name be 'RegisterFit')
  ```
  script:
    - if [[ -a .git/shallow ]]; then git fetch --unshallow; fi
    - julia -e 'using Pkg, LibGit2;
              user_regs = joinpath(DEPOT_PATH[1],"registries");
              mkpath(user_regs);
              all_registries = Dict("General" => "https://github.com/JuliaRegistries/General.git",
                            "HolyLabRegistry" => "git@github.com:HolyLab/HolyLabRegistry.git");
              Base.shred!(LibGit2.CachedCredentials()) do creds
                for (reg, url) in all_registries
                  path = joinpath(user_regs, reg);
                  LibGit2.with(Pkg.GitTools.clone(url, path; header = "registry $reg from $(repr(url))", credentials = creds)) do repo end
                end
              end'
    - julia -e 'using Pkg; Pkg.build(); Pkg.test("RegisterFit"; coverage=false)'
    ```
  - Assign your private ssh key which is paired with a public key in your Github account to the package in the Travis site.
    - Copies the contents of the private key ('id_rsa' file generated in the 'To use git protocol in GitHub' section - not 'id_rsa.pub') in the local machine to your clipboard.
    - Go to the setup page of the package in the Travis site you want to make to access this registry. (You can get there by choosing the package in your Travis repositories, clicking ‘More options’ button on the upper right corner and selecting ‘setting’ menu.)
    - Assign the private key in the clipboard to the ‘SSH Key’ field.
