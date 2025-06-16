### Smart `Ctrl+W` in Ghostty with Tmux

Tired of annoying confirmation dialogs when trying to close a terminal tab with running processes? Default terminal emulator behavior, even in Ghostty or iTerm2, often presents a clumsy 'Confirm close?' popup if anything is active. This configuration eliminates that friction, making your `Ctrl+W` key truly smart and context-aware.

Many "solutions" either universally block `Ctrl+W` for a terminal or kill everything ruthlessly. Neither is what a discerning user wants. This configuration provides precise, context-aware `Ctrl+W` behavior for your Ghostty setup, leveraging Tmux.

This setup ensures:

-   **`Ctrl+W` closes the Ghostty tab** when your `zsh` shell is the active foreground process.

-   **`Ctrl+W` is effectively ignored (blocked)** when any other application (like Neovim, SSH sessions, `htop`, `less`, etc.) is in the foreground. Crucially, because Tmux intercepts this key globally, applications like Neovim will **not**be able to use their own internal `Ctrl+W` commands (e.g., for buffer management or split navigation). This configuration prioritizes preventing accidental tab closure over internal application keybindings.

#### Setup

1.  **Ensure Tmux is installed.**

2.  **`~/.zshrc` Configuration** This block ensures that when you open a new Ghostty tab, it automatically launches into its own unique Tmux session. Place this at the **very top** of your `~/.zshrc` file to guarantee it runs first.

    ```
    TMUX_SESSION_NAME_PREFIX="ghostty-session"

    if [ -z "$TMUX" ]; then
        UNIQUE_SESSION_NAME="${TMUX_SESSION_NAME_PREFIX}-$(uuidgen | head -c 8)"
        exec tmux new-session -s "$UNIQUE_SESSION_NAME" -c "$(pwd)" -E
    fi

    ```

3.  **`~/.tmux.conf` Configuration** This is the core of the intelligent `Ctrl+W` binding. Place this in your Tmux configuration file, typically located at `/Users/yourusername/.tmux.conf`.

    ```
    bind-key -n C-w if-shell "test '#{pane_current_command}' = 'zsh'" {\
        kill-session\
        } {\
        run-shell "true"\
        }
    ```

4.  **Ghostty Configuration (`~/.config/ghostty/config.toml`)** Ghostty passes `Ctrl` key combinations to the application by default. Ensure you **do not** have any conflicting `Ctrl+W` bindings in your Ghostty configuration that would prevent the key from reaching Tmux. Remove any explicit `Ctrl+W` bindings if they exist.

    ```
    # No specific changes needed unless you have conflicting Ctrl+W bindings.
    # Ensure no entry like: "w" = { mods = "ctrl", action = "CloseTab" }
    keybind = ctrl+w=reload_config
    ```

#### How to Apply

1.  Save changes to `~/.zshrc` and `~/.tmux.conf`.

2.  **Close all existing Ghostty tabs completely.**

3.  Open a new Ghostty tab. This will initialize Tmux with the new configuration.

#### Testing

-   **At `zsh` prompt:** Press `Ctrl+W`. The Ghostty tab should close.

-   **With `nvim` running:** Launch `nvim`. Press `Ctrl+W`. Nothing should happen. (Neovim will not see `Ctrl+W` for its internal commands; Tmux intercepts and ignores it).

-   **With `ssh` running:** Launch `ssh user@host`. Press `Ctrl+W`. Nothing should happen.

-   **With `htop` running:** Launch `htop`. Press `Ctrl+W`. Nothing should happen.
