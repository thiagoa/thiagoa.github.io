require 'guard/plugin'

module ::Guard
  class JekyllServer < Plugin
    def start
      puts 'Starting Jekyll server...', '' if @pid.nil?
      @pid = Process.fork { `bundle exec jekyll serve` }
    end

    def stop
      Process.kill 'HUP', @pid
    end

    def reload
      puts '', '', 'Restarting Jekyll server...'
      stop and start
    end

    def run_all
      reload
    end

    def run_on_changes(paths)
      reload
    end
  end
end

guard 'jekyll_server' do
  watch %r{_posts/.+}
end
