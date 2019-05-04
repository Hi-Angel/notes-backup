# measuring time of a function

Code snippet to use:

```
        int64 before = GLib.get_monotonic_time();
        foo()
        int64 after = GLib.get_monotonic_time();
        stderr.printf(" took: %f ms\n", ((double)(after - before)) / 1000);
```

# Improve conversations loading time

First stack trace I got for an email being added, likely irrelevant:

```
#0    0x00005555556921ad in conversation_list_box_add_email (self=0x555557e16990, email=0x555557dc8b60, append_row=1)
    at ../src/client/conversation-viewer/conversation-list-box.vala:950
#1    0x000055555568d050 in conversation_list_box_load_conversation_co (_data_=0x5555572f1710)
    at ../src/client/conversation-viewer/conversation-list-box.vala:629
#2    0x000055555568ca31 in conversation_list_box_load_conversation (self=0x555557e16990, query=0x0, _callback_=0x5555556add66 <conversation_viewer_load_conversation_ready>, _user_data_=0x555555be3200)
    at ../src/client/conversation-viewer/conversation-list-box.vala:21
#3    0x00005555556ae382 in conversation_viewer_load_conversation_co (_data_=0x555555be3200) at ../src/client/conversation-viewer/conversation-viewer.vala:281
#4    0x00005555556adb92 in conversation_viewer_load_conversation (self=0x55555744a3a0, conversation=0x555557e38250, email_store=0x555557534150, avatar_store=0x5555571d81b0, _callback_=0x5555555af452 <______lambda127__gasync_ready_callback>, _user_data_=0x555555dd5280)
    at ../src/client/conversation-viewer/conversation-viewer.vala:13
#5    0x00005555555af9c6 in geary_controller_on_conversations_selected (self=0x555555be8130, selected=0x555557e44320)
    at ../src/client/application/geary-controller.vala:1311
#6    0x000055555559d136 in _geary_controller_on_conversations_selected_conversation_list_view_conversations_selected (_sender=0x55555743cc30, selected=0x555557e44320, self=0x555555be8130)
    at ../src/client/application/geary-controller.vala:287
#7    0x00007ffff7bede75 in g_closure_invoke + 0x1b5 () at /usr/lib/libgobject-2.0.so.0
#8    0x00007ffff7bdafd5 in No symbol matches 0x00007ffff7bdafd5. () at /usr/lib/libgobject-2.0.so.0
#9    0x00007ffff7bdf1ae in g_signal_emit_valist + 0xebe () at /usr/lib/libgobject-2.0.so.0
#10   0x00007ffff7be0080 in g_signal_emit + 0x90 () at /usr/lib/libgobject-2.0.so.0
#11   0x000055555567249c in conversation_list_view_do_selection_changed (self=0x55555743cc30)
    at ../src/client/conversation-list/conversation-list-view.vala:396
#12   0x000055555566e360 in _conversation_list_view_do_selection_changed_geary_idle_manager_idle_func (manager=0x55555729c420, self=0x55555743cc30)
    at ../src/client/conversation-list/conversation-list-view.vala:78
#13   0x00005555558d55bc in geary_idle_manager_execute (self=0x55555729c420) at ../src/engine/util/util-idle-manager.vala:130
#14   0x00005555558d595f in geary_idle_manager_handler_ref_execute (self=0x555557e14730) at ../src/engine/util/util-idle-manager.vala:53
#15   0x00005555558d5257 in _geary_idle_manager_handler_ref_execute_gsource_func (self=0x555557e14730) at util-idle-manager.c:245
#16   0x00007ffff7b00661 in g_main_context_dispatch + 0x161 () at /usr/lib/libglib-2.0.so.0
#17   0x00007ffff7b02739 in No symbol matches 0x00007ffff7b02739. () at /usr/lib/libglib-2.0.so.0
#18   0x00007ffff7b0277e in g_main_context_iteration + 0x2e () at /usr/lib/libglib-2.0.so.0
#19   0x00007ffff7461f5e in g_application_run + 0x21e () at /usr/lib/libgio-2.0.so.0
#20   0x0000555555593bf2 in _vala_main (args=0x7fffffffdf28, args_length1=1) at ../src/client/application/main.vala:33
#21   0x0000555555593c3b in main (argc=1, argv=0x7fffffffdf28) at ../src/client/application/main.vala:7
```

The `yield new_list.load_conversation(query);` gets executed relatively quickly, ≈0.5 seconds with disregard to conversations size. The rendering is likely being done somewhere else.

2nd stacktrace:

```
#0    0x0000555555691b27 in conversation_list_box_add_email (self=0x555557e59830, email=0x7fff44057aa0, append_row=0)
    at ../src/client/conversation-viewer/conversation-list-box.vala:901
#1    0x00005555556904c5 in conversation_list_box_finish_loading_co (_data_=0x55555802c030)
    at ../src/client/conversation-viewer/conversation-list-box.vala:830
#2    0x000055555568f712 in conversation_list_box_finish_loading_ready (source_object=0x555557e59830, _res_=0x7fff4c250cc0, _user_data_=0x55555802c030)
    at ../src/client/conversation-viewer/conversation-list-box.vala:809
#3    0x00007ffff7489dcc in No symbol matches 0x00007ffff7489dcc. () at /usr/lib/libgio-2.0.so.0
#4    0x00007ffff748a7a5 in No symbol matches 0x00007ffff748a7a5. () at /usr/lib/libgio-2.0.so.0
#5    0x0000555555690f13 in conversation_list_box_throttle_loading_co (_data_=0x7fff481aaa40)
    at ../src/client/conversation-viewer/conversation-list-box.vala:857
#6    0x0000555555690d4b in _conversation_list_box_throttle_loading_co_gsource_func (self=0x7fff481aaa40) at conversation-list-box.c:3518
#7    0x00007ffff7b00661 in g_main_context_dispatch + 0x161 () at /usr/lib/libglib-2.0.so.0
#8    0x00007ffff7b02739 in No symbol matches 0x00007ffff7b02739. () at /usr/lib/libglib-2.0.so.0
#9    0x00007ffff7b0277e in g_main_context_iteration + 0x2e () at /usr/lib/libglib-2.0.so.0
#10   0x00007ffff7461f5e in g_application_run + 0x21e () at /usr/lib/libgio-2.0.so.0
#11   0x0000555555593bf2 in _vala_main (args=0x7fffffffdf28, args_length1=1) at ../src/client/application/main.vala:33
#12   0x0000555555593c3b in main (argc=1, argv=0x7fffffffdf28) at ../src/client/application/main.vala:7
```

I pinned down lots of loading time to the foreach in `finish_loading`. For 21 mail it takes from 3 to 6 seconds to complete. For reference, the code snip:

        foreach (Geary.Email email in to_insert) {
            EmailRow row = add_email(email, false);
            // Since uninteresting rows are inserted above the
            // first expanded, adjust the scrollbar as they are
            // inserted so as to keep the list scrolled to the
            // same place.
            row.enable_should_scroll();
            row.should_scroll.connect(() => {
                    listbox_adj.value += GtkUtil.get_border_box_height(row);
                });

            // Only adjust for the loading row going away once
            loading_height = 0;

            yield row.view.load_avatar(this.avatar_store);
            yield throttle_loading();
        }

I measured every call here, and the only noticeable amounts were:

* `add_email` line took 7xx ms in total for 21 email; a line alone takes 25..50 ms.
* `yield throttle_loading()` takes a lot, 150..250 ms every call.

### Email construction

`ConversationMessage` constructor at `conversation-message.vala` takes most of the time in the `add_email`. Down the inside of `ConversationMessage` the reason is `ConversationWebView(config)`

# non-ASCII text attachments getting broken

`GMime.Part` is a "part" of a multipart MIME message. Also, they represented as a tree i Geary.

# Geary inserts newlines

Newlines are added when sending a mail. I think it happens at `ComposerPageState.htmlToText`. If not, one can trace it from `get_text()` at `composer-web-view.vala`.

# misc

"To compare" function in `ArrayList` at least in Geary is `equal_to`.

# performance of CoversationListView

`data.render` gets called a lot, but apparently I can't skip calls because that code is exactly what does the rendering.

# Reading Stacktrace

After vala gets translated to C, functions gets added and renamed. Some transformation examples:

* constructor Foo: `foo_construct`
* `new Foo()`: `foo_new` → `foo_construct`
* constructor `new Foo.something()`: `foo_new_something` → `foo_construct_something`
