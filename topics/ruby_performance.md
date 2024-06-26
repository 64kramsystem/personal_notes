# Ruby performance

- [Ruby performance](#ruby-performance)
  - [Collections](#collections)
    - [Array filling/clearing](#array-fillingclearing)
    - [Unique array values](#unique-array-values)
    - [Max value](#max-value)
  - [Garbage collection, with profiling helpers](#garbage-collection-with-profiling-helpers)
    - [Memory profiling](#memory-profiling)
      - [Using built-in APIs](#using-built-in-apis)
  - [Performance Profiling/Benchmarking](#performance-profilingbenchmarking)
    - [ruby-prof (tracing)](#ruby-prof-tracing)
    - [benchmark/ips](#benchmarkips)

## Collections

### Array filling/clearing

Methods performance (but test anyway):

- if fully resizing the array on each cycle, `#fill` is faster
- if not, `#clear` is faster
- `#new(size, value)` is relatively fast, but allocates

### Unique array values

DON'T use `Set.new(array)` - it's twice as slow as `array.uniq`.

### Max value

Using `[val1, val2].max` is as fast as using a conditional.

## Garbage collection, with profiling helpers

Garbage collection should always be stopped when measuring!

```rb
GC.disable
GC.enable
GC.start   # starts now
```

Profiling:

```rb
GC.count  # number of times the GC ran from VM start
GC.stat   # detailed stats

GC::Profiler.enable
yield
GC.start
GC::Profiler.report   # returns nil if it didn't run
GC::Profiler.disable
```

### Memory profiling

Don't use `memory_profiler`; based on one report, `ruby-prof` is more robust, and it includes allocations:

```rb
RubyProf.measure_mode = RubyProf::ALLOCATIONS
```

#### Using built-in APIs

```rb
def allocate_count(&block)
  GC.disable
  before = ObjectSpace.count_objects
  yield
  after = ObjectSpace.count_objects
  after.each { |k,v| after[k] = v - before[k] }
  after[:T_HASH] -= 1                           # exclude the just created hash
  after[:FREE] += 1                             # ^
  GC.enable
  after.reject { |k,v| v == 0 }
end
```

Other ObjectSpace functions:

```rb
puts ObjectSpace.each_object.count
puts ObjectSpace.each_object(Numeric).count
ObjectSpace.each_object(Complex) { |c| puts c }

# Adds other *very slow* methods
#
require "objspace"

ObjectSpace.count_objects_size
ObjectSpace.memsize_of("Sav")      # Can be inaccurate
ObjectSpace.memsize_of_all(String)
```

## Performance Profiling/Benchmarking

(Currently) GC pauses can't be detected by profilers.

Tracing profiles are slower (for dev), sampling ones are less accurate (for production).

For a sampling profiler, see `stackprof`.

### ruby-prof (tracing)

```rb
# Options:
#
# - WALL_TIME    : (default) since it includes I/O, other processes affecting I/O will increase this measure
# - CPU_TIME     : use when I/O is not relevant
# - PROCESS_TIME : generally the best, but doesn't include child processes
#
RubyProf.measure_mode = RubyProf::WALL_TIME # default

require 'ruby-prof'; p_result = RubyProf::Profile.profile do # ...

# Different outputs (with openers)
#
end; RubyProf::CallStackPrinter.new(p_result).print(File.open('/tmp/prof.htm', 'w'));                 `xdg-open /tmp/prof.htm`
end; RubyProf::GraphHtmlPrinter.new(p_result).print(File.open('/tmp/prof.htm', 'w'), min_percent: 5); `xdg-open /tmp/prof.htm`
end; RubyProf::CallTreePrinter.new(p_result).print(path: '/tmp', profile: 'prof');                    `qcachegrind /tmp/prof.callgrind*`
end; RubyProf::FlatPrinter.new(p_result).print(STDOUT)
```

With FlatPrinter:

- methods marked with `*` are recursively called.
- `%self`: time spent in this method (%)
- `total` = `self` + `child` (secs)

Quick and dirty terminal output:

```rb
buffer = StringIO.new
RubyProf::FlatPrinter.new(result).print(buffer)
puts buffer.
  string.
  lines.
  grep(/DumpAccountService/).        # filter by this class
  sort_by { |l| -l.split[1].to_f }.  # order by total (time spent in this method and its children)
  map { |l| l.sub(%r( \S+$), '') }.  # remove source location (which can be "(irb)"
  join
```

Convert a call tree output into a flamegraph:

```sh
g cl https://github.com/brendangregg/FlameGraph.git
./stackcollapse.pl /tmp/prof.callgrind.out        > /tmp/prof.callgrind.out.folded
./flamegraph.pl    /tmp/prof.callgrind.out.folded > /tmp/prof.flame.svg
```

### benchmark/ips

```rb
Benchmark.ips do |x|
  x.config(time: 1, warmup: 1) # default test time: 5, warmup: 2
  x.report("bsearch1") { bsearch1(array, rand(array_size)) }
  x.report("bsearch2") { bsearch2(array, rand(array_size)) }
  x.compare!
end; nil

# Calculating -------------------------------------
#             bsearch1 54.917k i/100ms
#             bsearch2 31.073k i/100ms
# -------------------------------------------------
#             bsearch1 782.667k (± 2.7%) i/s - 3.954M
#             bsearch2 374.304k (± 2.4%) i/s - 1.895M
# Comparison:
#             bsearch1: 782666.6 i/s
#             bsearch2: 374303.7 i/s - 2.09x slower
```

In order to avoid running the GC, must use a custom suite:

```rb
class GCSuite
  def warming(*); run_gc; end
  def running(*); run_gc; end
  def warmup_stats(*); end
  def add_report(*); end

  private

  def run_gc
    GC.enable
    GC.start
    GC.disable
  end
end

Benchmark.ips do |x|
  x.config(suite: GCSuite.new)
  # ...
end; nil
```
