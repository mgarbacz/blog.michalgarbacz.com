GITHUB_REPO = "mgarbacz/blog.michalgarbacz.com"

desc "Publish to gh-pages"
task :publish do

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
