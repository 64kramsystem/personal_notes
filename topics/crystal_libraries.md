# Crystal libraries

- [Crystal libraries](#crystal-libraries)
  - [Stdlib](#stdlib)
    - [IO](#io)
    - [OptionParser](#optionparser)
    - [HTTP/URI](#httpuri)
  - [Shards](#shards)
    - [Progress (Ruby: ProgressBar)](#progress-ruby-progressbar)

## Stdlib

### IO

```cr
File.read(file)                      # No IO.read
File.write(file, str)                # No IO.write
```

### OptionParser

```cr
def parse_commandline_arguments
  # As of Jull/2025, Crystal doesn't support passing union NamedTuples as kwargs, so we work this
  # problem around by predefining the variable, so that it stays a concrete type.
  opt_args = {
    myopt: "foo" # set this to the default defined in the invoked method
  }

  parser = OptionParser.new do |parser|
    parser.banner = "Usage: #{PROGRAM_NAME} [-h|--help] $myarg"

    parser.on("-h", "--help", "Show this help message") do
      puts parser
      exit
    end

    parser.on("-o MYOPT", "--myopt=MYOPT", "My option") do |value|
      # Variable is replaced with another of the same type.
      opt_args = opt_args.merge(myopt: value)
    end

    parser.invalid_option do |flag|
      STDERR.puts "Unknown option: #{flag}", parser
      abort
    end
  end

  case ARGV.size
  when 1
    pos_arg = ARGV[0]
  else
    STDERR.puts "Error: Only one argument is accepted!", parser
    abort
  end

  [opt_args, pos_arg]
end

def meth(pos_arg, myopt : String = "default"); end

opt_args, pos_arg = parse_commandline_arguments
meth(pos_arg, **opt_args)
```

### HTTP/URI

`URI.open` doesn't exist in Crystal.

```rb
# WATCH OUT! Redirections are not followed.
#
resp = HTTP::Client.get(addr)          # returns a HTTP::Client::Response; whose body is a String
HTTP::Client.get(address) { |resp| … } # passes an IO

resp = HTTP::Client.head(addr)         # same

resp.status.success?
resp.status.redirection?
resp.success?                          # shortcut
```

## Shards

### Progress (Ruby: ProgressBar)

This library has less functionality than the Ruby counterpart. This is a simple downloader class; the main addition is the adjustment of the width to the terminal:

```cr
class DownloadWithProgress
  # Rough space taken by the right-side summary (percent, sizes, ETA)
  SUMMARY_BUDGET = 40
  MIN_BAR        = 10

  def execute(address : String, destination_file : String)
    head = HTTP::Client.head(address)
    raise "File not found!" if !head.success?

    file_basename = File.basename(destination_file)
    total = head.headers["Content-Length"].to_i32
    bar_width, title = compute_bar_width(file_basename)
    theme = Progress::Theme.new(width: bar_width, bar_start: "#{title} [", bar_end: "]")

    HTTP::Client.get(address) do |response|
      File.open(destination_file, "wb") do |file|
        bar = Progress::IOBar.new(total: total, theme: theme)
        writer = bar.progress_writer
        IO.copy(response.body_io, IO::MultiWriter.new(file, writer))
        bar.finish! if !bar.done?
      end
    end
  end

  private def compute_bar_width(file_basename : String) : {Int32, String}
    cols = `tput cols`.to_i

    chrome = 3 # " [" + "]"
    title_max = cols - SUMMARY_BUDGET - MIN_BAR - chrome
    title_max = 0 if title_max < 0

    title = ellipsize(file_basename, title_max)

    bar_width = cols - SUMMARY_BUDGET - title.size - chrome
    bar_width = MIN_BAR if bar_width < MIN_BAR

    {bar_width, title}
  end

  private def ellipsize(string : String, max : Int32) : String
    return string if string.size <= max
    return "…" if max <= 1
    string.chars.first(max - 1).join + "…"
  end
end
```
