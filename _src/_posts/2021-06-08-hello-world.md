---
description: About my new website, featuring 170+ color schemes
tags: web
---

Here's a writeup on how i made this website

**Notice: this website has evolved quite a lot since this post, this might not be up to date**

I've been wanting to get into blogging random tech rants, maybe something useful too. Guides, maybe?

Well, i kept putting this off for quite a while, as i just couldn't decide which way to build it. I've considered the big frameworks, server side rendering, handwritting static assets, all the stuff.

Okay. So (as you can see) i got it done and i'm writing my first blog entry, about the blog itself.

How did i build it?


## Semantic HTML and 170+ CSS schemes
I decided to start building the [about page](../index.md) as a purely static page. I was looking for a good and pure CSS library i could use, as to keep the site as simple and lightweight as possible, considering i don't need any dynamic content.

First, i'm not a huge fan of utility CSS or heavily class based HTML. In my opinion, HTML should have as much semantic meaning as possible, and not be concerned with how a CSS sheet does the styling. I also love markdown, and i like having a 1:1 conversion from it to HTML, without having to add stuff. Blogging in markdown was big on the site wishlist.

Second, although they might be useful (specially when CSS did not support variables), i prefer to use CSS variables (root level custom properties) instead of preprocessors such as sass.

My reason for this is my aim of using [base16](https://github.com/chriskempson/base16) schemes to dynamically theme my website (170+ themes!) in a lazy way. That is, only loading any specific scheme on demand. Using a preprocessor would end up in generating full CSS sheets for each scheme option, which is not very bandwidth effective.

There's a plethora of different ways to offer schemes to a user. I did it by factoring out all colors on my base CSS sheet to variables, and using [flavours](https://github.com/misterio77/flavours) to generate a CSS sheet (with just the 16 color variables) for each existing scheme. These can be selected using a `input` field (check the palette button at the top), and will set a `link` tag's `href` value to the scheme, taking precedence over the default two schemes (one light and one dark, for `prefers-color-scheme: dark`) i specified on the base sheet.

With that done, i also set a tiny `localStorage` key to the browser, so the scheme preference can be persisted across visits.

Here's a code snippet:
```js
let chosen_scheme_css = document.getElementById("chosen-scheme");

// Set a given scheme
function setTheme(scheme) {
chosen_scheme_css.setAttribute(
  "href",
  "/assets/schemes/" + scheme + ".css"
);
localStorage.setItem("current-scheme", scheme);
}
// Reset scheme to defaults (rose-pine moon or dawn, depending on preference)
function resetTheme() {
chosen_scheme_css.removeAttribute("href");
localStorage.removeItem("current-scheme");
}

// Get stored scheme from localStorage and reapply it
let stored_scheme = localStorage.getItem("current-scheme");
if (stored_scheme) {
setTheme(stored_scheme);
}
```

This seems to me a very simple and clean way of doing it, without introducing complicated JS libraries to the equation. The colors even fade based on `animation` CSS property!

## A simple CSS styling

With that in mind, i set out to search for a minimal and semantic CSS sheet, preferably one using CSS variables.

Checking a few lists, i found [water.css](https://watercss.kognise.dev/), and it ticked all the boxes. It looks absolutely beautiful and is pure CSS.

All i had to do get my theming working was three `link` elements, which import, in order:
- Water.css
- My own sheet, which maps base0x variables to water.css ones, and provides two default schemes
- The user defined color scheme, once set (has an empty href by default)

## Code reuse and markdown pages

Okay, that's good enough for a personal website with a single page. But what if i want other pages with the same common elements, such as the navbar, the scheme dialog, footer... We would have to duplicate everything, ending up in a huge amount of duplicated HTML code, and a lot of pain to make changes everywhere at once.

Fetching content at runtime from a full fledged server and database is little overkill for a blog that i update once in a blue moon. When we don't need user generated data, adding content to the website at compile time makes a lot more sense. Enter Static Site Generation, or SSG.

Again, another problem of complete solutions is flexibility. I already have my styling and structure set, i just wanted a simple tool to factor out duplicated code. I stumbled upon [Jekyll](https://jekyllrb.com/) and its simplistic and modular goodness.

Jekyll is very straightforward and doesn't force you to change everything to support it. It works with templating (liquid) and markdown rendering. You can factor out common stuff from your website into includes (that work kinda like componentes) or layouts. All of this gets built into static assets, it doesn't get in your way and works great. It's also worth mentioning that most of the features are opt-in, you don't have to change absolutely nothing from your static assets if you don't want to.

## Automating style building

One of (the most useful, i think) my little projects is called [flavours](https://github.com/misterio77/flavours). It's a unixy CLI tool that downloads and builds color schemes and templates that follow the [base16](https://github.com/chriskempson/base16) standard.

It's mostly built for people that use heavily customizable desktop programs (terminals, window managers, status bars, etc). The ideia is setting up just once, and them instantly theming your entire setup with any base16 scheme, by replacing delimiters inside text configuration files (people call them dots). Schemes are really easy to make (flavours can even generate them from images, usually wallpapers), so this allows the power of infinite color schemes on a desktop.

It also works as a standard [base16 builder](https://github.com/chriskempson/base16/blob/master/builder.md), replacing scheme variables inside templates and outputting them. There's a CSS variables template, so it's really easy to generate a CSS sheet for hundreds of different schemes. Here's how that template looks:
{% raw %}
```css
/* {{scheme-name}} by {{scheme-author}} */
:root {
  --base00: #{{base00-hex}};
  --base01: #{{base01-hex}};
  --base02: #{{base02-hex}};
  --base03: #{{base03-hex}};
  --base04: #{{base04-hex}};
  --base05: #{{base05-hex}};
  --base06: #{{base06-hex}};
  --base07: #{{base07-hex}};
  --base08: #{{base08-hex}};
  --base09: #{{base09-hex}};
  --base0A: #{{base0A-hex}};
  --base0B: #{{base0B-hex}};
  --base0C: #{{base0C-hex}};
  --base0D: #{{base0D-hex}};
  --base0E: #{{base0E-hex}};
  --base0F: #{{base0F-hex}};
}
```
{% endraw %}
Pretty simple, huh? The spec provides those placeholders, which can be used in virtually any kind of file that specifies colors.

Here's how it looks with a color scheme (`pasque`) applied:
```css
/* Pasque by Gabriel Fontes (https://github.com/Misterio77) */
:root {
  --base00: #271C3A;
  --base01: #100323;
  --base02: #3E2D5C;
  --base03: #5D5766;
  --base04: #BEBCBF;
  --base05: #DEDCDF;
  --base06: #EDEAEF;
  --base07: #BBAADD;
  --base08: #A92258;
  --base09: #918889;
  --base0A: #804ead;
  --base0B: #C6914B;
  --base0C: #7263AA;
  --base0D: #8E7DC6;
  --base0E: #953B9D;
  --base0F: #59325C;
}
```

So, how do we generate one for every existing scheme? Additionally, there's a HTML tag `datalist` that allows for suggestions on an `input` form element. Maybe we can build one so the user knows which schemes are available?

Of course. flavours allows for great scriptability, so just use a bash script:
```bash
datalist_file="_includes/scheme-datalist.html"
template_path=~/.local/share/flavours/base16/templates/styles/templates/css-variables.mustache

# Add datalist opening tag
echo -n "<datalist id=\"scheme-list\">" > $datalist_file
# For each scheme
flavours list -l | while read slug; do
    # Get scheme file path
    scheme_path=$(flavours info $slug | head -1 | cut -d '@' -f2)
    # Build scheme
    flavours build $scheme_path $template_path > assets/schemes/$slug.css
    # Add entry to datalist
    echo -n "<option>$slug</option>" >> $datalist_file
done
# Add datalist closing tag
echo "</datalist>" >> $datalist_file
```

This script basically uses `flavours list` to get a list of all installed schemes and iterates over them. For each scheme, we build the `css-variables` template with it, outputting to the assets directory, and add its slug to the file with `<datalist>`.

## Wrapping up

Okay, this was my first blog entry, and that's how my simpl(ish) and minimalist site works. Feel free to check the [source code](https://github.com/misterio77/misterio-me) to learn more, or grab something for your own use (everything's MIT licensed).

Here's a few links to the stuff i used:
- [water.css](https://github.com/kognise/water.css) (base CSS styling)
- [prism.js](https://github.com/PrismJS/prism/) (code highlighting library)
- [jekyll](https://jekyllrb.com/) (static page generator)
- [base16](https://github.com/chriskempson/base16) (color theme spec)
- [flavours](https://github.com/misterio77/flavours) (base16 builder)
