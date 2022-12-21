# Latex

## Basics

- math expressions: surrounded with `$`
- comments: start with `%`

## Math chars/expressions

```latex
x^2  x^{2}          % exponentiation/superscript
x_1  x_{1}  E_{r,c} % subscript
\prime              % prime

\sqrt{x}            % square root
\vec{V}             % vector (overhead arrow)
\sum{x}             % summation (no limits)
\sum_{x=1}^{n}      % summation (with limits)
\frac{x}{y}         % fraction
\pm                 % ± (plus/minus)
\bar{x}             % top bar (average)
\cdot               % multiplication dot (e.g. dot product)

| \vert \lvert \rvert \left| \right| % vertical bar (magnitude/absolute) symbol
( ) \left(  \right) % round brackets
\langle  \rangle    % angle brackets
\alpha ...          % greek letters
\infty              % infinity

\cdot               % multiplication dot
\times              % multiplication cross

% matrices
% the *dots are filling dots

\begin{bmatrix}
  x_11   & \dots  & x_1n   \\
  \vdots & \ddots & \vdots \\
  x_d1   & x_d3   & x_dn
\end{bmatrix}

% Newlines are `\\` (and others, like `\newline`); it seems that in Anki/Mathjax 2, they require a
% wrapping block (see https://github.com/mathjax/MathJax/issues/2312).
%
\displaylines {     % cleanest block
  ax + by = c \\
  dx + ey = f
}
```
