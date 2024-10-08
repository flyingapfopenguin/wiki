---
title: "Julia"
date: 2023-04-06T22:54:01+02:00
---

## Installation

Clone the git repository https://github.com/JuliaLang/julia.git, checkout the desired version, install the dependencies
```bash
apt install make gcc g++ gfortran
```

and build Julia:
```bash
make clean; make -C deps uninstall && rm -rf ~/julia/usr && make
```

## Package Installation

### Installation as Julia packages

To install packages and develop versions of a packages run
```julia
]add <package>
]dev <package>
```

To update the installed packages use
```julia
]update
```
in the package manager.

### Using a cloned git repository

If the current working directory is a Julia packages, e.g. contains a `Project.toml`, we can use the package by running
```julia
using Pkg
Pkg.activate(".").
```

## Environments

Compare the [documentation of Pkg.jl](https://pkgdocs.julialang.org/v1/environments/).

Most important we can active profiles using
```julia
]activate [--shared] <profile>
```

## Tips & Tricks

### JLFzf

The Julia package [JLFzf](https://github.com/Moelf/JLFzf.jl) fzf support for searching your history.

### Startup Configuration

Here is an example startup configuration, that loads `Revise` and `JLFzf` and automatically loads packages in the current working directory:

```julia
ENV["JULIA_EDITOR"] = "vim"

# In presentations or non-interactive sessions do none of the following
if Base.isinteractive() && !isdefined(Main, :IJulia)

    # Load Revise
    atreplinit() do repl
        try
            @eval using Revise
        catch e
            @warn "Error initializing Revise" exception=(e, catch_backtrace())
        end
    end

    # Load JLFzf
    # This is just the example configuration of the project, see
    # https://github.com/Moelf/JLFzf.jl?tab=readme-ov-file#sample-startupjl
    import REPL
    import REPL.LineEdit
    import JLFzf
    const mykeys = Dict{Any,Any}(
        # primary history search: most recent first
        "^R" => function (mistate, o, c)
            line = JLFzf.inter_fzf(JLFzf.read_repl_hist(),
            "--read0",
            "--tiebreak=index",
            "--height=80%");
            JLFzf.insert_history_to_repl(mistate, line)
        end,
    )
    function customize_keys(repl)
        repl.interface = REPL.setup_interface(repl; extra_repl_keymap = mykeys)
    end
    atreplinit(customize_keys)
     
    # Automatically activate environment if Project.toml and Manifest.toml are present
    if isfile("Project.toml") && isfile("Manifest.toml")
        using Pkg
        @info "Auto-activating environment"
        Pkg.activate(".")
    end

end
```

### Managing Jupyter Kernel

List the installed julia kernel with
```bash
jupyter kernelspec list
```

Install new kernel from the julia shell via
```julia
using IJulia
IJulia.installkernel("<name your kernel>", "--project=<environment>")
```

und uninstall them with
```bash
jupyter kernelspec uninstall <kernel>
```
