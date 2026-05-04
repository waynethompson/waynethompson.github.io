# waynethompson.github.io

Built with Jekyll

## Setup (Linux)

Install Ruby and build dependencies:

```bash
sudo apt update
sudo apt install -y ruby-full build-essential zlib1g-dev
```

Install Bundler and Jekyll gems (without sudo, using local gem path):

```bash
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

gem install bundler jekyll
```

Install project dependencies:

```bash
bundle install
```

## Usage

Serve the site locally:

```bash
bundle exec jekyll serve -w --incremental
```

Update jekyll:

```bash
bundle update jekyll
```