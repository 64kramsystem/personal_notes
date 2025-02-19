# Crystal libraries

- [Crystal libraries](#crystal-libraries)
  - [Stdlib](#stdlib)
    - [OptionParser](#optionparser)

## Stdlib

### OptionParser

```cr
def parse_commandline_arguments
  exit_with_status = nil

  parser = OptionParser.new do |opts|
    opts.banner = "Usage: #{PROGRAM_NAME} [-h|--help] $log_file $minutes $max_stats"

    opts.on("-h", "--help", "Show this help message") do
      exit_with_status = 0
    end

    opts.invalid_option do |flag|
      STDERR.puts "Unknown option: #{flag}", ""
      exit_with_status = 1
    end
  end

  if ARGV.size != 3
    STDERR.puts "Error: Unexpected number of arguments!", ""
    exit_with_status = 1
  end

  if exit_with_status
    puts parser
    exit(exit_with_status.not_nil!)
  end

  ARGV
end
```
