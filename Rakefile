GITHUB_REPO = "mgarbacz/blog.michalgarbacz.com"

task :build do
  system "jekyll build"
end

task :watch do
  system "jekyll server --watch"
end

task :publish => [:build] do

  Dir.chdir "_public" do
    system "git init"
    system "git checkout --orphan gh-pages"
    system "git add ."
    system "git commit -m 'Site updated at #{Time.now.utc}'"
    system "git remote add origin git@github.com:#{GITHUB_REPO}.git"
    system "git push origin gh-pages --force"
  end

  Dir.chdir ".."
end
