## Structure

```rust
src --- main.rs
     +- model.rs
     +- ui.rs
```

## main.rs

```rust
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
```

- Below is a default code using Crossterm as backend.

```rust
fn main() -> Result<(), Box<dyn Error>> {
  // setup terminal
  enable_raw_mode()?;
  let mut stdout = io::stdout();
  execute!(stdout, EnterAlternateScreen, EnableMouseCapture)?;
  let backend = CrosstermBackend::new(stdout);
  let mut terminal = Terminal::new(backend)?;

  // create app and run it
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
```

- In below, until `terminal.draw(|f| ui(f, &mut app))?;` is also default. Event handling part would be modified.

```rust
fn run_app<B: Backend>(
  terminal: &mut Terminal<B>,
  mut app: App,
) -> io::Result<()> {
  loop {
		// "fn ui" comes from ui.rs
	  terminal.draw(|f| ui(f, &mut app))?;
    
	  if let Event::Key(key) = event::read()? {
	    if key.kind == KeyEventKind::Press {
	      match key.code {
	        KeyCode::Char('q') => return Ok(()),
					// "app" is a structure defined in model.rs
	        KeyCode::Left => app.items.unselect(),
	        KeyCode::Down | KeyCode::Char('j') => app.items.next(),
	        KeyCode::Up | KeyCode::Char('k') => app.items.previous(),
	        _ => {}
				}
      }
    }
  }
}
```

## model.rs

```rust
use ratatui::widgets::*;

/// Original comment(https://github.com/ratatui-org/ratatui/blob/main/examples/list.rs)
/// This struct holds the current state of the app. In particular, it has the `items` field which is
/// a wrapper around `ListState`. Keeping track of the items state let us render the associated
/// widget with its state and have access to features such as natural scrolling.
///
/// Check the event handling at the bottom to see how to change the state on incoming events.
/// Check the drawing logic for items on how to specify the highlighting style for selected items.
pub struct StatefulList<T> {
  pub state: ListState,
  pub items: Vec<T>,
}

impl<T> StatefulList<T> {
  pub fn with_items(items: Vec<T>) -> StatefulList<T> {
    StatefulList {
      state: ListState::default(),
      items,
    }
  }

  pub fn next(&mut self) {
    let i = match self.state.selected() {
      Some(i) => {
        if i >= self.items.len() - 1 {
          0
        } else {
          i + 1
        }
      }
      None => 0,
    };
    self.state.select(Some(i));
  }

  pub fn previous(&mut self) {
    let i = match self.state.selected() {
      Some(i) => {
        if i == 0 {
          self.items.len() - 1
        } else {
          i - 1
        }
      }
      None => 0,
    };
    self.state.select(Some(i));
  }

  pub fn unselect(&mut self) {
    self.state.select(None);
  }
}

pub struct App<'a> {
  pub items: StatefulList<(&'a str, usize)>,
}

impl<'a> App<'a> {
  pub fn new() -> App<'a> {
    App {
      items: StatefulList::with_items(vec![
        ("Item0", 1),
        ("Item1", 2),
        ("Item2", 1),
        ("Item3", 3),
        ("Item4", 1),
        ("Item5", 4),
        ("Item6", 1),
        ("Item7", 3),
        ("Item8", 1),
        ("Item9", 1),
        ("Item10", 1),
      ]),
    }
  }
}
```

## ui.rs

```rust
use crate::App;
use ratatui::{prelude::*, widgets::*};

pub fn ui(f: &mut Frame, app: &mut App) {
  // Create two chunks one for list, another for keymap
  let chunks = Layout::default()
    .direction(Direction::Vertical)
    .constraints([Constraint::Min(3), Constraint::Length(3)])
    .split(f.size());

  // Iterate through all elements in the `items` app and append some debug text to it.
  let items: Vec<ListItem> = app
    .items
    .items
    .iter()
    .map(|i| {
      let mut lines = vec![Line::from(i.0)];
      for _ in 0..i.1 {
        lines.push(
        "lorem.."
          .italic()
          .into(),
        );
      }
      ListItem::new(lines).style(Style::default().fg(Color::White))
      })
      .collect();

    // Create a List from all list items and highlight the currently selected one
  let items = List::new(items)
    .block(Block::default().borders(Borders::ALL).title("Selectable List"))
    .highlight_style(
      Style::default()
        .bg(Color::LightGreen)
        .add_modifier(Modifier::BOLD),
        )
        .highlight_symbol(">> ");

  // We can now render the item list
  f.render_stateful_widget(items, chunks[0], &mut app.items.state);

  // Bottom_bar
  let keys = String::from("q = Quit / j = Down / k = Up");

  let bottom_bar = Paragraph::new(keys)
    .alignment(Alignment::Center)
    .fg(Color::White);
  f.render_widget(bottom_bar, chunks[1])
}
```
## Result
![Result](https://github.com/shacaranda/ratatui-study/assets/8787660/a91eee90-4dbc-4da3-8d5b-c9e736aa99d3)
