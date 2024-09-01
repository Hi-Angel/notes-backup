# slots'n'signals

Declared via like `slots:`, `signals:`. It's a macros that's supposedly parsed by Qt MOC compiler.

There's a function `connect()` to connect them. It's also possible that predefined slots does not require `connect()`ing *(needs confirmation)*.

# Debugging

## GammaRay

An introspection tool for Qt GUI. Can be attached to processes.

Tabs on the left:

* `Objects` has widgets tree, and their properties.
* `Widgets` allows to select widgets and see their preview. Also: the preview has toolbox, which has `picker` tool. Unfortunately you can't click with this on the process itself, but instead you can "pick" a widget in the preview.

## Printing events

Doesn't seem to be possible without modification of the code. Modifications basically come down to:

1. Create a class that derives from the one that whose event you want to print, and replace in code all instances of the latter with the former. Check that it compiles.
2. Add an `override` function that overrides `event()` function of the `QObject`.
3. *(optional)* make a pretty printer since there seems not to be one for events.

Example:

```c++
#include <QApplication>
#include <QLabel>
#include <QScrollArea>
#include <QScroller>
#include <QDebug>
#include <QMetaEnum>

void gen_label_text(QLabel& label) {
    const char sentence[] = "'th sentence here\n";
    QString text;
    for (int line = 0; line < 1e3; ++line) {
        text += QString::number(line) + sentence;
    }
    label.setText(text);
}

/// Gives human-readable event type information.
// Credits to https://stackoverflow.com/a/22535470/2388257
QDebug operator<<(QDebug str, const QEvent * ev) {
   static int eventEnumIndex = QEvent::staticMetaObject
         .indexOfEnumerator("Type");
   str << "QEvent";
   if (ev) {
      QString name = QEvent::staticMetaObject
            .enumerator(eventEnumIndex).valueToKey(ev->type());
      if (!name.isEmpty()) str << name; else str << ev->type();
   } else {
      str << (void*)ev;
   }
   return str.maybeSpace();
}

struct MyClass : public QScrollArea {
    bool event(QEvent* ev) override {
        qDebug() << "handling an event " << ev;
        return QScrollArea::event(ev);
    }
};

int main(int argc, char **argv) {
    QApplication app (argc, argv);
    QLabel label;
    gen_label_text(label);
    MyClass qsa;
    qsa.setWidget(&label);

    QScroller *scroller = QScroller::scroller(&label);
    QScroller::grabGesture(qsa.viewport(), QScroller::TouchGesture);

    qsa.show();
    return app.exec();
}
```

# QtCreator WebAssembly/emsdk configuration

1. Download `emsdk`. Note that `emscripten` isn't it. `emsdk` contains similarly named script, which will do the installation as follows:
   1. `emsdk install latest`
   2. `emsdk activate latest`
2. QtCreator: go to `Help → About Plugins…` and activate webassembly plugin.
3. QtCreator: go to `Edit → Prefrences → Devices` section, switch to `WebAssembly` tab, and add the path to `emsdk` root directory. Note that emask installation commands will download stuff to subdirs, but you ignore that and point at the root emsdk dir.
4. TODO

# Misc

* Adding Qt versions: `Edit → Preferences → Kits`, tab `Qt Versions`, button `Add…`: it expects path to `qmake` executable. So e.g. on Archlinux there are `qmake` and `qmake6` binaries provided by `qt5-base` and `qt6-base` packages accordingly, that's what the package expects.
