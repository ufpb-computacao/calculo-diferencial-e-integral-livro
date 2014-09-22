require 'net/http'
require 'rake/clean'
require 'find'
require 'date'
require 'open3'

task :default => [:wip]

SOURCE_FILES = FileList['livro/livro.asc', 'livro/capitulos/*']
@RELEASE_DIR = 'releases/current'
@BOOK_SOURCE_DIR = 'livro'
@BOOK_SOURCE = 'livro/livro.asc'
@BOOK_TARGET = 'livro/livro.pdf'
@A2X_BIN = '~/ambiente/asciidoc/a2x.py'
WIP_ADOC = "#{@BOOK_SOURCE_DIR}/wip.adoc"
RELEASE_BOOK_SOURCE = "#{@RELEASE_DIR}/#{@BOOK_SOURCE_DIR}/livro.asc"
RELEASE_BOOK  = "#{@RELEASE_DIR}/#{@BOOK_SOURCE_DIR}/livro.pdf"
RELEASE_WIP_ADOC =  "#{@RELEASE_DIR}/#{@BOOK_SOURCE_DIR}/wip.adoc"
RELEASE_WIP_PDF  =  "#{@RELEASE_DIR}/#{@BOOK_SOURCE_DIR}/wip.pdf"
OPEN_PDF_CMD=`git config --get producao.pdfviewer`.strip
A2X_COMMAND="-v -k -f pdf --icons -a docinfo1 -a edition=`git describe` -a lang=pt-BR -d book --dblatex-opts '-T computacao -P latex.babel.language=brazilian' -a livro-pdf"
PROJECT_NAME = File.basename(Dir.getwd)
LIVRO_URL = `git config --get livro.url`.strip
GITHUB_REPO = `git config remote.origin.url`.strip.gsub('git@github.com:','').gsub('.git','')

directory @RELEASE_DIR

CLEAN.include('releases')

desc "Sync, build and open wip file"
task :wip => [WIP_ADOC, "sync", "wip:build", "wip:open"]
task :edit => ["wip:edit"]


namespace "wip" do

  desc "Create new wip file from book source"
  task "new" do
    cp "#{@BOOK_SOURCE}", "#{@BOOK_SOURCE_DIR}/wip.adoc"
  end

  file WIP_ADOC do
    Rake::Task["wip:new"].invoke
  end

  file RELEASE_WIP_PDF do
    system "#{@A2X_BIN} #{A2X_COMMAND} #{@RELEASE_DIR}/#{@BOOK_SOURCE_DIR}/wip.adoc"
  end

  desc "Open wip pdf"
  task :open => RELEASE_WIP_PDF do |t|
      puts "#{OPEN_PDF_CMD} #{@RELEASE_DIR}/#{@BOOK_SOURCE_DIR}/wip.pdf"
      system "#{OPEN_PDF_CMD} #{@RELEASE_DIR}/#{@BOOK_SOURCE_DIR}/wip.pdf"
  end

  desc "Open docbook xml from wip build"
  task "xml" do
    system "#{OPEN_PDF_CMD} #{@RELEASE_DIR}/#{@BOOK_SOURCE_DIR}/wip.xml"
  end

  desc "Edit wip source"
  task "edit" do
    system "gvim #{WIP_ADOC}"
  end

  desc "build book from #{@RELEASE_DIR}"
  task :build => [WIP_ADOC, :sync] do
    system "#{@A2X_BIN} #{A2X_COMMAND} #{@RELEASE_DIR}/#{@BOOK_SOURCE_DIR}/wip.adoc"
  end

end




desc "Archive, build and open book file"
task :book => [:clean, :archive, "book:build", "book:open"]

namespace "book" do

  desc "Build book"
  task :build => ['sync'] do
    system "#{@A2X_BIN} #{A2X_COMMAND} #{@RELEASE_DIR}/#{@BOOK_SOURCE}"
  end

  desc "Open pdf book"
  task "open" do
    system "#{OPEN_PDF_CMD} #{@RELEASE_DIR}/#{@BOOK_TARGET}"
  end

  desc "Open docbook xml from book build"
  task "xml" do
    system "#{OPEN_PDF_CMD} #{@RELEASE_DIR}/#{@BOOK_SOURCE_DIR}/livro.xml"
  end

  desc "Edit book source"
  task "edit" do
    system "gvim #{@BOOK_SOURCE}"
  end

  desc "Release new edition book"
  task :release, [:tag] do |t, args|
    #PROJECT = sh "`git config --get remote.origin.url | cut -f 2 -d / | cut -f 1 -d .`"
    puts "PROJECT_NAME='#{PROJECT_NAME}' LIVRO_URL='#{LIVRO_URL}'"
    mkdir_p "~/releases/#{PROJECT_NAME}"
    cd "~/releases/#{PROJECT_NAME}"
    `wget #{LIVRO_URL}`
    puts "Salvando arquivo em #{Dir.getwd}"
    mv "livro.pdf", "#{PROJECT_NAME}-#{args.tag}.pdf"
    #Dir.mkdir(File.join(Dir.home, ".foo"), 0700)
  end

end

desc "Extract files from repository (git archive)"
task :archive => :clean do
  system "git archive --format=tar --prefix=#{@RELEASE_DIR}/ HEAD | (tar xf -) "
end

desc "Local sync of the files"
task :sync => @RELEASE_DIR do |t|
  system "rsync -r --delete #{@BOOK_SOURCE_DIR}/ #{@RELEASE_DIR}/#{@BOOK_SOURCE_DIR}"
end

namespace "tag" do

  desc "List project tags"
  task :list do
    sh "git tag --list"
  end

  desc "Aplly a tag to the project. The tag can be used as the edition."
  task :apply, [:tag] do |t, args|
    sh "git status"
    sh "git tag -a #{args.tag} -m 'Gerando versão #{args.tag}'"
  end

  desc "Delete a tag applied."
  task :delete, [:tag] do |t,args|
    sh "git tag -d #{args.tag}"
  end

  desc "Push tags"
  task "push" do
    sh "git push origin --tags"
  end

  desc "Generate revision history, compare HEAD and tag.
  The tag is optional, if not specified it will use the last tag applied."
  task :revision, [:tag] do |t, args|
    last_tag = `git describe --abbrev=0`.strip
    args.with_defaults(:tag => last_tag)
    tag = args.tag
    edition = ENV.fetch('edition', "#{tag} FIXME!")
    authors = ENV.fetch('authors', " FIXME! ")
    date = Date.today.strftime "%d/%m/%Y"
    history, s = Open3.capture2("git log --reverse --format='- %s. ' #{args.tag}..HEAD")
    revision = "\n
    <revision>
      <revnumber>#{edition}</revnumber>
      <date>#{date}</date>
      <authorinitials>#{authors}</authorinitials>
      <revremark>
#{history}      </revremark>
    </revision>\n\n"
    puts revision
  end

  desc "Open docinfo for edition.
  Before apply tag you should edit docinfo and add the revision history."
  task :docinfo do
    puts "#{OPEN_PDF_CMD} #{@RELEASE_DIR}/#{@BOOK_SOURCE_DIR}/wip.pdf"
    system "xdg-open #{@BOOK_SOURCE_DIR}/docinfo.xml"
  end

end


desc "Open orginal pdf to work"
task :original do
    sh "#{OPEN_PDF_CMD} original/original.pdf"
end


namespace "config" do

#  desc "Configure open command. `xdg-open` for ubuntu and open for `osx`"
  task :pdfviewer, [:app] do |t,args|
    sh "git config --global producao.pdfviewer #{args.app}"
  end

end


desc "Download new Rakefile"
task :uprake do
  `wget --output-document=Rakefile https://raw.githubusercontent.com/edusantana/novo-livro/master/Rakefile`
end


desc "Build images from R files"
task :r
task :sync => :r

FileList['livro/images/**/*.R'].each do |source|
  rpdf = source.ext('pdf')
  file rpdf => source do |t|
    rm_rf "Rplots.pdf"
    sh "R --no-save < #{t.source}"
    mv "Rplots.pdf","#{t.name}"
  end
  task :r => rpdf
end

desc "Build images from dot files"
task :dot
task :sync => :dot

FileList['livro/images/**/*.dot'].each do |source|
  epsfile = source.ext('eps')
  file epsfile => source do |t|
    sh "dot -Teps -o #{t.name} #{t.source}"
  end
  task :dot => epsfile
end

namespace "github" do
  desc "List issues from github milestone"
  task :issues, [:milestone] do |t,args|
    puts "Acessing: #{GITHUB_REPO} milestone=#{args.milestone}"
    require 'octokit'
#    require 'highline/import'
    client = Octokit::Client.new
    milestone = nil
    milestones = client.list_milestones(GITHUB_REPO, state: "all")
    opcoes = milestones.map {|m| m[:title]}

    if (args.milestone) then
      #puts "milestones: #{milestones}"
      milestones.each do |m|
        if m[:title] == args.milestone then
          milestone = m
        end
      end
    else
      milestone = milestones[-1]
    end
    puts "Milestone: #{milestone[:title]}"

    puts ""
    puts "Para adicionar ao docinfo.xml:\n"
    issues = client.list_issues(GITHUB_REPO, state:'Closed', milestone:milestone[:number], direction:'asc')
    issues.each do |i|
      puts "<ulink url=\"{gitrepo}/issues/#{i[:number]}\">#{i[:title]};</ulink>"
    end
    puts ""
    puts "Para adicionar ao release notes no github:"
    issues.each do |i|
      puts "- #{i[:title]} (##{i[:number]});"
    end

  end
end
