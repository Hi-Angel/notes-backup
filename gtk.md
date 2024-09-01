# WEB frontend in GTK

Can be done with "broadway" server shipped with GTK. Works by running e.g. server as `gtk4-broadwayd -p 8085` and then app as `GDK_BACKEND=broadway gnome-calculator`. I tested it with standard apps like `gnome-editor` and `gnome-calculator`.

Note:

* an app is displayed as a draggable window with a bar at the top. It should be possible to modify an app to get rid of the header and to be always maximized, but I didn't test that.
* As of 29 Aug 2024 its "broadway" state is experimental, quoting IRC:

> ```
> [14:21:32] <mclasen> a good reminder that we should document broadway more clearly as an experiment
> [14:26:10] <mclasen> it was written as a one-off experiment, and is not actively developed
> [14:26:37] <mclasen> it works and supported the gtk 4.0 features, but it is falling behind newer developments
> [14:37:14] <ebassi> […]: what mclasen said. Broadway is an experiment
> [14:37:25] <ebassi> if you want to use it, you get to fix it
> ```

# test app with a text widget (tested with gtk3)

test.cpp:

```
#include <gtk/gtk.h>

int main(int argc, char** argv) {
	GtkBuilder      *builder;
	GtkWidget       *window;

	gtk_init(&argc, &argv);

	builder = gtk_builder_new();
	gtk_builder_add_from_file (builder, "test.glade", NULL);

	window = GTK_WIDGET(gtk_builder_get_object(builder, "window_main"));
	gtk_builder_connect_signals(builder, NULL);
    g_signal_connect(G_OBJECT(window), "destroy", gtk_main_quit, NULL);

	g_object_unref(builder);

	gtk_widget_show(window);
	gtk_main();

	return 0;
}
```

test.glade:
```
<?xml version="1.0" encoding="UTF-8"?>
<!-- Generated with glade 3.22.1 -->
<interface>
  <requires lib="gtk+" version="3.20"/>
  <object class="GtkWindow" id="window_main">
	<property name="can_focus">False</property>
	<child>
	  <placeholder/>
	</child>
	<child>
	  <object class="GtkEntry">
		<property name="visible">True</property>
		<property name="can_focus">True</property>
	  </object>
	</child>
  </object>
</interface>
```

Compile as `g++ test.cpp -o a $(pkg-config --cflags --libs gtk+-3.0)`.

# Misc

## Undo

### Misc

I've been linked to this page https://wiki.gnome.org/Projects/GTK/Undo Haven't found anything interesting though. A lot of functional being discussed I don't see as something needed right now, and can be added later by someone if they see it useful.

* Undo/redo GTK enablement:
    * GTK3: Opt-out by default, but opt-in if Ctrl+z already bound.
    * GTK4: Opt-out by default.

```
    -- implement for widgets and stuff
    class Undoable widget where
    undoStack :: [undoAtom]
    undoAtom :: undoAtom -> widget -> widget
    redoAtom :: undoAtom -> widget -> widget

```

### todo

* create undo/redo stack on GtkEntry init, and save it with it.
* wire up on-character-typed action for GtkEntry (don't save it to the undo stack every time though — good for CPU cache)
* wire up Ctrl+z action for GtkEntry
* wire up Ctrl+Shift+z action for GtkEntry
* profit!

# Input Methods

Can be overriden with `GTK_IM_MODULE` env. variable. Built-in GTK IMs are `gtk-im-context-none` and `gtk-im-context-simple`.

* `gtk-im-context-none` supposedly with this should still Compose key be working, since it's supposed to be processed by xkbcommon lib. But for some reason it completely stops working.
