  
## Markdown / Mermaid offline editor  
  
The idea is to use IntelliJ Idea Community edition with Markdown and Mermaid plugins as an offline cross-platform Markdown document manager and visualization tool.    
Which totally worked for me, on MacOS and Linux.    
Also, storing Markdown documents on GitHub and using all the power of `git` to manage versions / editions is a piece of cake now.   
  
---  
### Markdown plugin  
  
Markdown plugin comes bundled with IntelliJ Idea.  
  
---  
  
### Installing 'Mermaid' plugin  
  
1. Install an IDE if you don’t have one:  
  
-> IntelliJ IDEA Community  
  
2. Open your IDE and press  ⌘ ,  to open the IDE settings.  
  
3. Select  **Plugins**, click  and then click  **Install Plugin from Disk**.  
  
4. Select the plugin archive file and click  **OK**.  
  
5. Click  **OK**  to apply the changes and restart your IDE if prompted.  
  
Plugin: https://plugins.jetbrains.com/plugin/20146-mermaid    
Beware the compatibility issues.    
Plugin `Mermaid-0.0.14_IJ.231.zip` works with `ideaIC-2023.1.5.tar.gz`  
  
---  
  
### Here's how to change font size in Markdown view:  
  
`CMD` + `,` (Settings)    
-> Languages and Frameworks    
    -> Markdown    
        -> Custom CSS    
            -> CSS rules  
  
<script>alert("hey!")</script>  
  
  
```  
body {  
    font-size: 90% !important;  
}  
```  
  
Font size 90% works for me!  
  
---  
  
### Jetbrains documentation on Custom CSS  
[jetbrains.com link](https://www.jetbrains.com/help/idea/markdown.html#css)  
  
IntelliJ IDEA provides default style sheets for rendering HTML in the preview pane. These style sheets were designed to be consistent with the default  [UI themes](https://www.jetbrains.com/help/idea/user-interface-themes.html). You can configure specific CSS rules to make small presentation changes (for example, change the font size for headings or line spacing in lists) or you can provide an entirely new CSS to better match your expected output (for example, if you want to replicate the  [GitHub Markdown style](https://github.com/sindresorhus/github-markdown-css)).  
  
![Markdown preview with a custom CSS that resembles GitHub rendering style](https://resources.jetbrains.com/help/img/idea/2023.2/markdown-preview-custom-css.png "Markdown preview with a custom CSS that resembles GitHub rendering style")  
  
### Configure CSS for rendering HTML preview  
  
1.  Press  ⌘Сmd0,  to open the IDE settings and then select  Languages & Frameworks | Markdown.  
  
2.  Configure the settings under  Custom CSS:  
  
    -   Select  Load from  to specify the location of a custom CSS file.  
  
    -   Select  CSS rules  rules to enter specific CSS rules that you want to override.  
  
3.  Click  OK  to apply the changes.  
