# mdx_math_svg

Python-Markdown extension to render equations as embedded SVG.  
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

## Installation

### Dependencies

Requires Python 3.6 or newer (I presume, to be tested) and Python-Markdown 3.0 or newer, as
well as a LaTeX installation. If you don't have LaTeX, I recommend [TeX Live](https://tug.org/texlive/).

The extension uses the `latex` and `dvisvgm` commands, which should both be installed with your LaTeX
distribution. Make sure these programs are on the path so that they can be found.

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
See below for a list of all configuration options. This does not allow you to use the SVG cache,
use the preferred method below instead.

### Preferred method

The preferred method allows you to use the cache. The cache will avoid rendering the same LaTeX
equations over and over again every time you rebuild your Markdown content. Only new or changed
equations will be rendered.

```python
import markdown
import mdx_math_svg

# Create Python-Markdown instance
math_svg_extension = mdx_math_svg.MathSvgExtension(inline_class='math', display_class='math'),
md = markdown.Markdown(extensions=[math_svg_extension])

# Load cache (if it's there)
math_cache_file_name = 'math_cache'
mdx_math_svg.load_cache(math_cache_file_name)

# Use Python-Markdown instance
md.convert("My Markdown text")

# Save cache
mdx_math_svg.save_cache(math_cache_file_name)
```

Load the cache file before you begin any Markdown parsing, and save the cache file when you're done with
all of the conversion. Any SVG that was in the cache but was not needed this session will be deleted from
the cache. Therefore, you should never re-use the cache among different projects, each project should have
its own cache.

### Configuration parameters

The extension recognizes the following parameters:

- `inline_class`: Inline math is SVG wrapped in a `<span>` tag, this option adds a class name to it.
  Defaults to the empty string.
- `display_class`: Display math is SVG wrapped in a `<div>` tag, this option adds a class name to it. - Default: ''"
  Defaults to the empty string.
- `smart_dollar`: Use MathSvg's smart dollars. Defaults to `True`. See below for an explanation.
- `block_syntax`: This is an array that chooses which of the block syntaxes are recognized:
  "dollar" (`$$...$$`), "square" (`\[...\]`), and/or "begin" (`\begin{env}...\end{env}`).
  Defaults to `['dollar', 'square', 'begin']`.
- `inline_syntax`: This is an array that chooses which of the inline syntaxes are recognized:
  "dollar" (`$...$`), and/or "round" (`\(...\)`).
  Defaults to `['dollar', 'round']`.
- `fontsize`: Font size in em for rendering LaTeX equations. Defaults to 1, matching surrounding text,
  usually. Depending on the font used, you might want to increase or decrease this value a bit.

There are other things that can be configured with a bit more difficulty. The `mdx_math_svg.params`
variable is a struct containing the following:
```python
mdx_math_svg.params = {
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
    'dvisvgm_cmd': 'dvisvgm --no-fonts --exact',
    'libgs': None,
}
```
By changing these values, it is possible to change the preamble, and the options passed to `latex` and `dvisvgm`.

The `libgs` value contains the path to the Ghostscript library. If it cannot be found through the `LIBGS` environment
variable, you can manually set it.

### Formatting equations

We'll assume that you know how to write equations in LaTeX. This extension loads the amsmath, amsfonts and
amssymb packages in LaTeX.

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
