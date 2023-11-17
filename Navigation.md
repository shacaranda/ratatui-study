## Structure
src --- main.rs
     +- model.rs
     +- ui.rs
     +- ui --- sub1.rs
            +- sub2.rs
            +- sub3.rs
## main.rs
``` rust
mod ui;
mod model;

use crate::ui::*;
use crate::model::*;

use std::{
    error::Error,
    io,
};
use crossterm::{
    event::{self, DisableMouseCapture, EnableMouseCapture, Event, KeyCode, KeyEventKind},
    execute,
    terminal::{disable_raw_mode, enable_raw_mode, EnterAlternateScreen, LeaveAlternateScreen},
};
use ratatui::prelude::*;

fn main() -> Result<(), Box<dyn Error>> {
    // setup terminal
    enable_raw_mode()?;
    let mut stdout = io::stdout();
    execute!(stdout, EnterAlternateScreen, EnableMouseCapture)?;
    let backend = CrosstermBackend::new(stdout);
    let mut terminal = Terminal::new(backend)?;

    // create app and run it
    // let tick_rate = Duration::from_millis(250);
    let app = App::new();
    let res = run_app(&mut terminal, app);

    // restore terminal
    disable_raw_mode()?;
    execute!(
        terminal.backend_mut(),
        LeaveAlternateScreen,
        DisableMouseCapture
    )?;
    terminal.show_cursor()?;

    if let Err(err) = res {
        println!("{err:?}");
    }

    Ok(())
}

fn run_app<B: Backend>(
    terminal: &mut Terminal<B>,
    mut app: App,
) -> io::Result<()> {
    loop {
        terminal.draw(|f| ui(f, &mut app))?;
        if let Event::Key(key) = event::read()? {
            if key.kind == KeyEventKind::Press {
                match key.code {
                    KeyCode::Char('q') => return Ok(()),
                    KeyCode::Left => app.items.unselect(),
                    KeyCode::Down | KeyCode::Char('d') => app.sub1(),
                    KeyCode::Up | KeyCode::Char('c') => app.sub2(),
                    KeyCode::Char('l') => app.sub3(),
                    _ => {},
                }
            }
        }
    }
}
```
## model.rs
``` rust
use ratatui::widgets::*;

/// This struct holds the current state of the app. In particular, it has the `items` field which is
/// a wrapper around `ListState`. Keeping track of the items state let us render the associated
/// widget with its state and have access to features such as natural scrolling.
///
/// Check the event handling at the bottom to see how to change the state on incoming events.
/// Check the drawing logic for items on how to specify the highlighting style for selected items.
pub enum Views {
    Sub1,
    Sub2,
    Sub3,
}

pub struct App<'a> {
    pub view: Views,
}

impl<'a> App<'a> {
    pub fn new() -> App<'a> {
        App {
            view: Views::Detail,
        }
    }

    pub fn sub1(&mut self) {
        self.view = Views::Sub1; 
    }

    pub fn sub2(&mut self) {
        self.view = Views::Sub2;
    }

    pub fn sub3(&mut self) {
        self.view = Views::Sub3;
    }
}

```
## ui.rs
``` rust
mod sub1;
mod sub2;
mod sub3;

use sub1::*;
use sub2::*;
use sub3::*;
use crate::model::{App, Views};
use ratatui::prelude::*;

pub fn ui(f: &mut Frame, app: &mut App) {
    match app.view {
        Views::Sub1 => sub1(f, app),
        Views::Sub2 => sub2(f, app),
        Views::Sub3 => sub3(f, app),
    };
}
```
* The above pattern matching `match app.view` is the key code for navigation.
## ui/sub1.rs
``` rust
use crate::App;
use ratatui::{prelude::*, widgets::*};

pub fn sub1(f: &mut Frame, app: &mut App) {
    let greeting = Paragraph::new("hello world!");
    f.render_widget(greeting, f.size());
}
```
