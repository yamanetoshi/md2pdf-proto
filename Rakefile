require 'fileutils'
require 'rake/clean'

def use_bundler()
  if system("bundle > /dev/null 2>&1")
    "bundle exec "
  else
    ""
  end
end

def build(mode, chapter)
  sh "#{use_bundler}review-compile --target=#{mode} --footnotetext --singledirmode --stylesheet=style.css #{chapter} > tmp"
  mode_ext = {
    "html" => "html",
    "text" => "txt",
    "latex" => "tex",
    "idgxml" => "xml",
    "inao" => "inao"
  }
  FileUtils.mv "tmp", chapter.gsub(/re\z/, mode_ext[mode])
end

def build_all(mode)
  sh "#{use_bundler}review-compile --all --target=#{mode} --footnotetext --singledirmode --stylesheet=style.css"
end

["html", "text", "idgxml"].each{|mode|
  desc "build #{mode} (Usage: rake build re=target.re)"
  task mode.to_sym do
    if ENV['re'].nil?
      puts "Usage: rake build re=target.re"
      exit
    end
    build(mode, ENV['re'])
  end

  desc "build all #{mode}"
  task "#{mode}_all".to_sym do
    build_all(mode)
  end
}

MD2RE = "md2review"

SRCS = FileList["**/*.md"]
RESRCS = SRCS.ext('re')

TARGETNAME = "example"

desc 'generate review sources'
task "md2review" => RESRCS

rule '.re' => '.md' do |t|
  sh "#{MD2RE} #{t.source} > #{t.name}"
end

task :default => :pdf

desc 'generate PDF and EPUB file'
task :all => [:pdf, :epub]

desc 'generate PDF file'
task :pdf => "#{TARGETNAME}.pdf"

desc 'generate EPUB file'
task :epub => "#{TARGETNAME}.epub"

desc 'generate html file'
task :pandoc => "#{TARGETNAME}.html"

file "#{TARGETNAME}.html" => ["#{TARGETNAME}.md"] do
  sh "rm -f #{TARGETNAME}.html"
  sh "pandoc -f markdown -t html5 --template default.html5 -o #{TARGETNAME}.html #{TARGETNAME}.md"
end

SRC = FileList['*.re'] +  ["config.yml"]

file "#{TARGETNAME}.pdf" => RESRCS do
  sh "rm -f #{TARGETNAME}.pdf"
  sh "rm -rf #{TARGETNAME} #{TARGETNAME}-pdf"
  sh "#{use_bundler}review-pdfmaker config.yml"
end

file "#{TARGETNAME}.epub" => RESRCS do
  sh "rm -f #{TARGETNAME}.epub"
  sh "rm -rf #{TARGETNAME} #{TARGETNAME}-epub"
  sh "#{use_bundler}review-epubmaker config.yml"
end

CLEAN.include(["#{TARGETNAME}", "#{TARGETNAME}-pdf", "#{TARGETNAME}.pdf", "#{TARGETNAME}-epub", "#{TARGETNAME}.epub", RESRCS])
Dir["#{File.dirname(__FILE__)}/*.re"].sort.each do |path|
  ["txt", "html", "xml"].each{|ext|
    file_path = "#{File.dirname(__FILE__)}/#{File.basename(path, '.re')}.#{ext}"
    if File.exists? file_path
      CLEAN.include("#{File.basename(path, '.re')}.#{ext}")
    end
  }
end
