# coding: UTF-8
require 'rake/clean'
require 'redcarpet'

def load_index(filename)
	entries = IO.readlines(filename)
	entries.map {|e|
		e.chomp.gsub(/\s+#.*/, '')
	}
end

$index = load_index("index.list")

def make_index(target, sources)
	chapters = $index.map {|s|
		if s == "index.md"
			title = "目次"
		else
			title = open(s).readline.chomp
		end
		[s, title]
	}
	listfile = File.open(sources[0], "w")
	mdfile = File.open(target, "w")
	mdfile << "目次\n====\n"
	chapters.each {|ch|
		listfile << sprintf("%-20s #%s\n", ch[0], ch[1].slice(0..20))
		mdfile << "1. [#{ch[1]}](#{ch[0]})\n" unless ch[0] == "index.md"
	}
	listfile.close
	mdfile.close
end

class DocRenderer < Redcarpet::Render::XHTML
	@title = nil
	@source = nil

	attr_accessor :source, :title

	def doc_header
		str = "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n"
		str << "<!DOCTYPE html>\n<html>\n<head>\n"
		str << "<link rel=\"stylesheet\" href=\"default.css\" type=\"text/css\" />\n"
		str << "<title>#{@title}</title>" if @title
		str << "</head>\n<body>\n"
	end

	def doc_footer
		return "</body></html>\n" if @source == "index.md"
		str = ""
		inum = $index.index(@source)
		abort "#{@source} is not listed in the index file." if inum.nil?
		if inum > 0
			prevtitle = open($index[inum - 1]).readline.chomp
			str << "<p>前章: <a href=\"#{$index[inum - 1].gsub(/md$/,'html')}\">#{prevtitle}</a></p>"
		end
		if inum < $index.length - 1
			nexttitle = open($index[inum + 1]).readline.chomp
			str << "<p>次章: <a href=\"#{$index[inum + 1].gsub(/md$/,'html')}\">#{nexttitle}</a></p>"
		end
		str << "<p><a href=\"index.html\">目次</a></p>"
		str << "</body></html>\n"
	end

	def link(link, title, content)
		str = "<a href=\"#{link.sub(/\.md$/, '.html')}\""
		str << "title=\"#{title}\"" if title
		str << ">#{content}</a>"
	end
end

md = Redcarpet::Markdown.new(DocRenderer.new(:with_toc_data=>true))

MD_files = FileList["*.md"] - ["index.md"]
HTML_files = MD_files.ext('html') + ["index.html"]
CLEAN << HTML_files

task :default => [:build]

desc "Build all HTML files from markdown files."
task :build => HTML_files

file "index.md" => ["index.list"] + MD_files do |t|
	make_index(t.name, t.prerequisites)
end

rule ".html" => [".md"] do |t|
	source = open(t.source).readlines
	md.renderer.title = source[0].chomp
	md.renderer.source = t.source
	source = source.join
	File.open(t.name, 'w') {|dest|
		dest << md.render(source)
	}
end
