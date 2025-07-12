# Crystal libraries

- [Crystal libraries](#crystal-libraries)
  - [Stdlib](#stdlib)
    - [OptionParser](#optionparser)

## Stdlib

### OptionParser

```cr
def parse_commandline_arguments
  myopt: String? = nil

  parser = OptionParser.new do |parser|
    parser.banner = "Usage: #{PROGRAM_NAME} [-h|--help] $log_file $minutes"

    parser.on("-h", "--help", "Show this help message") do
      puts parser
      exit
    end

    parser.on("-o MYOPT", "--myopt=MYOPT", "My option") do |myopt|
      myopt = myopt
    end

    parser.invalid_option do |flag|
      STDERR.puts "Unknown option: #{flag}", parser
      abort
    end
  end

  if ARGV.size != 2
    STDERR.puts "Error: Unexpected number of arguments!", parser
    abort
  end

  [myopt, *ARGV]
end
```
