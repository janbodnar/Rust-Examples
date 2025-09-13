# Egui

## Command palette

```rust
use chrono::{DateTime, Local};

#[derive(Default)]
struct MyApp {
    show_command_palette: bool,
    command_input: String,
    status_message: String,
}

impl eframe::App for MyApp {
    fn update(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame) {
        // Handle keyboard shortcuts
        if ctx.input(|i| {
            i.modifiers.ctrl && i.modifiers.shift && i.key_pressed(egui::Key::P)
        }) {
            self.show_command_palette = true;
        }

        // Status bar at the bottom
        egui::TopBottomPanel::bottom("status_bar").show(ctx, |ui| {
            ui.horizontal(|ui| {
                if self.status_message.is_empty() {
                    ui.label("Ready...");
                } else {
                    ui.label(&self.status_message);
                }
            });
        });

        // Command Palette (VS Code style - at the top)
        if self.show_command_palette {
            egui::TopBottomPanel::top("command_palette").show(ctx, |ui| {
                ui.horizontal(|ui| {
                    ui.label(">");
                    let response = ui.add_sized(
                        [ui.available_width() - 20.0, 20.0], // Full width minus some padding
                        egui::TextEdit::singleline(&mut self.command_input)
                    );
                    response.request_focus();

                    if ui.input(|i| i.key_pressed(egui::Key::Enter)) {
                        self.execute_command(&self.command_input.clone(), ctx);
                        self.command_input.clear();
                        self.show_command_palette = false;
                    }

                    if ui.input(|i| i.key_pressed(egui::Key::Escape)) {
                        self.command_input.clear();
                        self.show_command_palette = false;
                    }
                });
            });
        }

        egui::CentralPanel::default().show(ctx, |ui| {
            // Position the quit button in the top-left corner
            ui.with_layout(egui::Layout::top_down(egui::Align::LEFT), |ui| {
                if ui.add_sized([80.0, 30.0], egui::Button::new("Quit")).clicked() {
                    // Close the application
                    ctx.send_viewport_cmd(egui::ViewportCommand::Close);
                }
            });
        });
    }
}

impl MyApp {
    fn execute_command(&mut self, command: &str, ctx: &egui::Context) {
        match command.to_lowercase().as_str() {
            "quit" | "exit" => {
                ctx.send_viewport_cmd(egui::ViewportCommand::Close);
            },
            "time" => {
                let now: DateTime<Local> = Local::now();
                self.status_message = format!("Current time: {}", now.format("%Y-%m-%d %H:%M:%S"));
            },
            "help" => {
                self.status_message = "Available commands: quit/exit, time, help".to_string();
            },
            _ => {
                self.status_message = format!("Unknown command: {}", command);
            }
        }
    }
}

fn main() -> Result<(), eframe::Error> {
    let options = eframe::NativeOptions {
        viewport: egui::ViewportBuilder::default()
            .with_inner_size([550.0, 450.0]),
            centered: true,
        ..Default::default()
    };

    eframe::run_native(
        "App",
        options,
        Box::new(|_cc| Box::new(MyApp::default())),
    )
}
```
