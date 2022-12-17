~~~
<!-- PlutoStaticHTML.Begin -->
<!--
    # This information is used for caching.
    [PlutoStaticHTML.State]
    input_sha = "493f1df8a4c98bf40721a792bac95c497e516298c618d7a288a5fd010041613c"
    julia_version = "1.8.3"
-->




~~~
+++
title = "Benchmarking"
+++

~~~


<div class="markdown"><h1>Benchmarking</h1>
<p>Este notebook contiene output de los macros <code>@benchmark</code> and <code>@code_warntype</code>. Ambos se muestran con <code>with_terminal</code> que viene de <a href="https://github.com/JuliaPluto/PlutoUI.jl">PlutoUI.jl</a>, véase abajo.</p>
<p>Definimos una función <code>double</code> y un diccionario de tipos para mostrar los problemas de &quot;type inference&quot; &#40;inferencia de tipos&#41; via <code>@code_warntype</code>:</p>
</div>

<pre class='language-julia'><code class='language-julia'>numbers = Dict(:one =&gt; 1f0, :two =&gt; 2.0)</code></pre>
<pre id='var-numbers' class='code-output documenter-example-output'>Dict{Symbol, AbstractFloat} with 2 entries:
  :two => 2.0
  :one => 1.0</pre>

<pre class='language-julia'><code class='language-julia'>function double(mapping, key::Symbol)
    return 2 * mapping[key]
end;</code></pre>



<div class="markdown"><p>Now, the code works.</p>
</div>

<pre class='language-julia'><code class='language-julia'>double(numbers, :one)</code></pre>
<pre id='var-hash458071' class='code-output documenter-example-output'>2.0f0</pre>

<pre class='language-julia'><code class='language-julia'>double(numbers, :two)</code></pre>
<pre id='var-hash234834' class='code-output documenter-example-output'>4.0</pre>


<div class="markdown"><p>But the <code>@code_warntype</code> shows some big red warnings:</p>
</div>

<pre class='language-julia'><code class='language-julia'>using PlutoUI: with_terminal</code></pre>


<pre class='language-julia'><code class='language-julia'>with_terminal() do
    @code_warntype double(numbers, :one)
end</code></pre>
<pre id="plutouiterminal">
MethodInstance for Main.var\"workspace#4\".double(::Dict{Symbol, AbstractFloat}, ::Symbol)
  from double(mapping, key::Symbol) in Main.var\"workspace#4\" at /home/runner/work/JuliaParaGenteConPrisa.jl/JuliaParaGenteConPrisa.jl/tutorials/benchmarking.jl#==#3f0e2049-8597-4dac-b499-4d7a8a35978e:1
Arguments
  #self#[36m::Core.Const(Main.var\"workspace#4\".double)[39m
  mapping[36m::Dict{Symbol, AbstractFloat}[39m
  key[36m::Symbol[39m
Body[91m[1m::Any[22m[39m
[90m1 ─[39m %1 = Base.getindex(mapping, key)[91m[1m::AbstractFloat[22m[39m
[90m│  [39m %2 = (2 * %1)[91m[1m::Any[22m[39m
[90m└──[39m      return %2
</pre>



<div class="markdown"><p>We can fix this by forcing all elements in the dictionary to have the same type. Specifically, to we force all elements to be of type <code>Float64</code>:</p>
</div>

<pre class='language-julia'><code class='language-julia'>typednumbers = Dict{Symbol, Float64}(:one =&gt; 1f0, :two =&gt; 2.0)</code></pre>
<pre id='var-typednumbers' class='code-output documenter-example-output'>Dict{Symbol, Float64} with 2 entries:
  :two => 2.0
  :one => 1.0</pre>


<div class="markdown"><p>This gets rid of all the type warnings:</p>
</div>

<pre class='language-julia'><code class='language-julia'>with_terminal() do
    @code_warntype double(typednumbers, :one)
end</code></pre>
<pre id="plutouiterminal">
MethodInstance for Main.var\"workspace#4\".double(::Dict{Symbol, Float64}, ::Symbol)
  from double(mapping, key::Symbol) in Main.var\"workspace#4\" at /home/runner/work/JuliaParaGenteConPrisa.jl/JuliaParaGenteConPrisa.jl/tutorials/benchmarking.jl#==#3f0e2049-8597-4dac-b499-4d7a8a35978e:1
Arguments
  #self#[36m::Core.Const(Main.var\"workspace#4\".double)[39m
  mapping[36m::Dict{Symbol, Float64}[39m
  key[36m::Symbol[39m
Body[36m::Float64[39m
[90m1 ─[39m %1 = Base.getindex(mapping, key)[36m::Float64[39m
[90m│  [39m %2 = (2 * %1)[36m::Float64[39m
[90m└──[39m      return %2
</pre>



<div class="markdown"><p>And makes the method more quick:</p>
</div>

<pre class='language-julia'><code class='language-julia'>using BenchmarkTools</code></pre>


<pre class='language-julia'><code class='language-julia'>with_benchmark_terminal() do
    @benchmark double(numbers, :one)
end</code></pre>
<pre id="plutouiterminal">
BenchmarkTools.Trial: 10000 samples with 990 evaluations.
 Range (min … max):  42.929 ns …  2.100 μs  ┊ GC (min … max): 0.00% … 96.73%
 Time  (median):     46.667 ns              ┊ GC (median):    0.00%
 Time  (mean ± σ):   49.169 ns ± 34.967 ns  ┊ GC (mean ± σ):  1.21% ±  1.67%

             ▃▁▆▂▇█▂                                           
  ▁▁▁▁▂▂▂▃▆▆▇███████▆▃▂▂▂▂▂▃▃▃▃▃▃▃▂▂▂▂▂▂▂▂▂▂▃▃▄▅▆▇▇▆▇▄▄▃▃▂▂▂▁ ▃
  42.9 ns         Histogram: frequency by time        55.6 ns <

 Memory estimate: 16 bytes, allocs estimate: 1.
</pre>


<pre class='language-julia'><code class='language-julia'>with_benchmark_terminal() do
    @benchmark double(typednumbers, :one)
end</code></pre>
<pre id="plutouiterminal">
BenchmarkTools.Trial: 10000 samples with 996 evaluations.
 Range (min … max):  23.092 ns …  2.314 μs  ┊ GC (min … max): 0.00% … 97.88%
 Time  (median):     24.398 ns              ┊ GC (median):    0.00%
 Time  (mean ± σ):   26.318 ns ± 34.997 ns  ┊ GC (mean ± σ):  1.72% ±  1.38%

   ▂▁▄▆▅▇█▆▇▄▅▃ ▂▂ ▁ ▁▁ ▁▁ ▁ ▂▁ ▁               ▂▃▁▂▁▂▁ ▁▁ ▁  ▂
  ▇███████████████████████████████▆▇▅▇▇▆▇▆▅▄▄▆█▇█████████████ █
  23.1 ns      Histogram: log(frequency) by time      32.7 ns <

 Memory estimate: 16 bytes, allocs estimate: 1.
</pre>



<div class="markdown"><h2>Appendix</h2>
</div>

<pre class='language-julia'><code class='language-julia'>function with_benchmark_terminal(f)
    out = sprint(show, "text/plain", f())
    with_terminal() do
        print(out)
    end
end;</code></pre>

<div class='manifest-versions'>
<p>Built with Julia 1.8.3 and</p>
BenchmarkTools 1.3.1<br>
PlutoUI 0.7.37
</div>

<!-- PlutoStaticHTML.End -->
~~~