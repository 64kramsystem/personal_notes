# Crystal libraries

- [Crystal libraries](#crystal-libraries)
  - [Stdlib](#stdlib)
    - [OptionParser](#optionparser)

## Stdlib

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
