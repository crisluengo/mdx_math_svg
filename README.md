# mdx\_math\_svg

[Python-Markdown](https://python-markdown.github.io) extension to render equations as embedded SVG.  
No MathJax, no images. Real vector drawings.

The resulting static pages load fast and will not reflow as equations are formatted.

The Markdown syntax recognized is:
```text
$Equation$, \(Equation\)

$$
    Display Equation
$$

\[
    Display Equation
\]

\begin{align}
    Display Equations
\end{align}
```

### Examples

These are some examples of the use of this Markdown plugin:

- My blog, prepared with [Pelican](https://blog.getpelican.com) and Python-Markdown, has some math-heavy posts,
  for example the post "[A simple implementation of snakes](https://www.crisluengo.net/archives/217)".

- The DIPlib documentation, prepared with [dox++](https://crisluengo.github.io/doxpp/), uses this plugin to
  generate pages like [this one](https://diplib.org/diplib-docs/why_tensors.html) and function documentation like
  [this ones](https://diplib.org/diplib-docs/nonlinear.html#dip-PeronaMalikDiffusion-dip-Image-CL-dip-Image-L-dip-uint--dip-dfloat--dip-dfloat--dip-String-CL)
  and the ones below it.

## Installation

### Dependencies

Requires Python 3.6 or newer (I presume, to be tested) and Python-Markdown 3.0 or newer, as
well as a LaTeX installation. If you don't have LaTeX, I recommend [TeX Live](https://tug.org/texlive/).

The extension uses the `latex` and `dvisvgm` commands, which should both be installed with your LaTeX
distribution. Make sure these programs are on the path so that they can be found. If you install only a
subset of the TeX Live distribution, you will likely need to add the following LaTeX packages using
the `tlmgr` tool: `standalone`, `preview`, `ucs`, `newtx`, `fontaxes`, and `xstring`.

### Install from PyPI

```bash
pip install mdx_math_svg
```

### Install locally

```bash
./setup.py build
./setup.py install
```

## Usage

### Quick and simple

The simple way to add the extension to the Python-Markdown parser is as follows:
```python
import markdown

extension_configs = {
    'mdx_math_svg': {
        'inline_class': 'math',
        'display_class': 'math'
    }
}
md = markdown.Markdown(extensions=['mdx_math_svg'], extension_configs=extension_configs)
```
See below for a list of all configuration options. To use the cache, load it before processing your Markdown
content, and save it after:
```python
import mdx_math_svg

# Load cache (if it's there)
math_cache_file_name = 'math_cache'
mdx_math_svg.load_cache(math_cache_file_name)

# Use Python-Markdown instance
md.convert("My Markdown text")

# Save cache
mdx_math_svg.save_cache(math_cache_file_name)
```

### Preferred method

Python-Markdown documentation says that the preferred method for loading a plugin is to create an instance
of the plugin and pass it into the Markdown parser, as follows:
```python
import markdown
import mdx_math_svg

# Create Python-Markdown instance
math_svg_extension = mdx_math_svg.MathSvgExtension(inline_class='math', display_class='math'),
md = markdown.Markdown(extensions=[math_svg_extension])

# Load cache (if it's there)
math_cache_file_name = 'math_cache'
math_svg_extension.latex2svg.load_cache(math_cache_file_name)

# Use Python-Markdown instance
md.convert("My Markdown text")

# Save cache
math_svg_extension.latex2svg.save_cache(math_cache_file_name)
```

### Using the cache

The cache will avoid rendering the same LaTeX equations over and over again every time you rebuild your
Markdown content. Only new or changed equations will be rendered.

Any SVG that was in the cache but was not needed this session will be deleted from the cache.
Therefore, you should never re-use the cache among different projects, each project should have its own cache.

Load the cache file before you begin any Markdown parsing, and save the cache file when you're done with
all of the conversion, as shown in the code snippets above. Note that the two code snippets use a different
syntax for loading and saving. `math_svg_extension.latex2svg.load_cache()` calls `mdx_math_svg.load_cache()`.
There is only one global cache, so it is shared among all instances of the plugin. Programs like Pelican
create a new instance of the plugin for each page they parse, hence the need for a globally-shared cache.

### Usage in Pelican

[Pelican](https://blog.getpelican.com) is a static site generator, a great way to publish a blog. If you choose
to use it with Markdown formatted input, you can use this plugin to add great-looking equations to your
blog.

To enable the plugin, add a line to your `MARKDOWN` extension configuration setting in `pelicanconf.py` with
the setting for the `mdx_math_svg` plugin. You should already have this variable declared, all you need to
do is add the one line:
```python
MARKDOWN = {
    'extension_configs': {
        'markdown.extensions.codehilite': {'css_class': 'highlight'},
        'markdown.extensions.extra': {},
        'markdown.extensions.meta': {},
        'mdx_math_svg': {'inline_class': 'math', 'display_class': 'math'},  # Add this line
    },
    'output_format': 'html5',
}
```

Next, create a file in your Pelican directory called `plugins/mdx_math_svg_cache.py`, and copy the following
into it:
```python
import mdx_math_svg
from pelican import signals

_filename = 'math_cache'

def load_cache(pelican):
    mdx_math_svg.load_cache(_filename)

def save_cache(pelican):
    mdx_math_svg.save_cache(_filename)

def register():
    signals.initialized.connect(load_cache)
    signals.finalized.connect(save_cache)
```

In your `pelicanconf.py` file, add the following:
```python
PLUGIN_PATHS = ["plugins"]
PLUGINS = ["mdx_math_svg_cache"]
```
You might already be using plugins. In this case, simply add the `mdx_math_svg_cache` plugin to the list.

The plugin code contains the string `'math_cache'`. This is the name of the cache file. Feel free to
change its name and/or location.

This cache doesn't interact well with the cache that Pelican uses. Since it purges equations not processed
during the current session, only equations used during the last run of Pelican will be kept. Pages cached
by Pelican won't be processed by this plugin, and hence their equations will be purged from the cache.

### Configuration parameters

The extension recognizes the following parameters:

- `inline_class`: Inline math is SVG wrapped in a `<span>` tag, this option adds a class name to it.
  Defaults to `'math'`.
- `display_class`: Display math is SVG wrapped in a `<div>` tag, this option adds a class name to it.
  Defaults to `'math'`.
- `smart_dollar`: Use mdx\_math\_svg's smart dollars. Defaults to `True`. See below for an explanation.
- `block_syntax`: This is an array that chooses which of the block syntaxes are recognized:
  "dollar" (`$$...$$`), "square" (`\[...\]`), and/or "begin" (`\begin{env}...\end{env}`).
  Defaults to `['dollar', 'square', 'begin']` (all three syntaxes are recognized).
- `inline_syntax`: This is an array that chooses which of the inline syntaxes are recognized:
  "dollar" (`$...$`), and/or "round" (`\(...\)`).
  Defaults to `['dollar', 'round']` (both syntaxes are recognized).
- `fontsize`: Font size in em for rendering LaTeX equations. Defaults to 1, matching surrounding text,
  usually. Depending on the font used, you might want to increase or decrease this value a bit.
- `additional_preamble`: LaTeX commands to add to the LaTeX document preamble. Can be used to load
  additional packages, for example. Either a string or an array of strings. Defaults to `[]`.  
  For example, I like to use `'additional_preamble': [r'\usepackage{nicefrac}']` to enable the `\nicefrac` command.

There are other things that can be configured with a bit more difficulty. The `math_svg_extension.latex2svg.params`
variable is a struct containing the following:
```python
math_svg_extension.latex2svg.params = {
    'fontsize': 1,  # em (in the sense used by CSS)
    'template': r"""
\documentclass[12pt,preview]{standalone}
{{ preamble }}
\begin{document}
\begin{preview}
{{ code }}
\end{preview}
\end{document}
""",
    'preamble': r"""
\usepackage[utf8x]{inputenc}
\usepackage{amsmath}
\usepackage{amsfonts}
\usepackage{amssymb}
\usepackage{newtxtext}
\usepackage{newtxmath}
""",
    'latex_cmd': 'latex -interaction nonstopmode -halt-on-error',
    'dvisvgm_cmd': 'dvisvgm --no-fonts --exact',  # Has '--precision=2' added if the dvisvgm version is >= 2.2.2
    'libgs': None,  # Gets set through environment variable or default location
}
```
By changing these values, it is possible to change the preamble, and the options passed to `latex` and `dvisvgm`.

The `libgs` value contains the path to the Ghostscript library. If it cannot be found through the `LIBGS` environment
variable, you can manually set it.

**TODO:** Add extension parameters to modify these values.

### Formatting equations

We'll assume that you know how to write equations in LaTeX. This extension loads the `amsmath`, `amsfonts` and
`amssymb` packages in LaTeX. Additional packages can be loaded through the `additional_preamble` setting.

Inline equations can (by default) be enclosed in single dollar signs (`$x$`) or escaped parenthesis
(round brackets, `\(x\)`). Note that the dollar signs are "smart" (by default), in that it is required for them
to touch the equation on both sides. That is, no spaces are allowed in between the dollar signs and the contained
equation. `$ x$` or `$x $` will not be recognized as an equation. This is to prevent sentences such as
"I owe you $1.75 and you owe me $2.50" to be incorrectly recognized as inline equations. This is configured
with the `smart_dollar` parameter.

Block equations can (by default) be enclosed in double dollar signs (`$$xxx$$`), escaped square brackets (`\[xxx\]`),
or any of the LaTeX environments that start math mode, for example `\begin{align} xxx \end{align}`. Block equations
must have one empty line before and one after, and cannot have any empty lines inside. There should be nothing
before or after the enclosing markers, except for an attribute list at the end if the standard `attr_list`
extension is enabled.

## License

Copyright 2021 by Cris Luengo   
Distributed under the MIT license (see [LICENSE](LICENSE)).

Based on:

- Extension logic, recognizing Markdown syntax:  
  Arithmatex from [PyMdown Extensions](https://github.com/facelessuser/pymdown-extensions)  
  Copyright 2014 - 2017 Isaac Muse <isaacmuse@gmail.com>  
  MIT license.

- Converting LaTeX into SVG:  
  latex2svg.py  
  Copyright 2017, Tino Wagner  
  MIT license.

- Tweaking the output of latex2svg for embedding into HTML5, and use of cache:  
  latex2svgextra.py from [m.css](https://github.com/mosra/m.css)  
  Copyright 2017, 2018, 2019, 2020 Vladimír Vondruš <mosra@centrum.cz>  
  MIT license.
