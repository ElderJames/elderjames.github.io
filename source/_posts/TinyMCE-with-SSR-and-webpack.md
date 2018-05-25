---
title: (转) TinyMCE with SSR and webpack
permalink: 'TinyMCE-with-SSR-and-webpack'
date: 2018-05-25 12:59:28
tags:
- Angular
categories:
- Angular
---

 - custom.d.ts

```ts
interface NodeRequire {
    ensure: (paths: string[], callback: (require: NodeRequireFunction) => void) => void
}

```
- texteditor.component.html

```html
<div *ngIf="ClientSide; then client else server"></div>

<ng-template #client>
  <div *ngIf="!TinyMCELoaded">
    <p>Loading...</p>
  </div>
  <div [attr.hidden]="TinyMCELoaded ? null : ''">
    <form (ngSubmit)="handleSubmit()">
      <textarea id="{{elementId}}">{{text}}</textarea>
    </form>
    <div *ngIf="!Saved">
      <p>Don't forget to save!</p>
    </div>
  </div>
</ng-template>

<ng-template #server>
  <div>
    <p>Loading...</p>
  </div>
</ng-template>
```

- texteditor.component.ts

```ts

import {
    Component,
    AfterViewInit,
    EventEmitter,
    OnDestroy,
    Input,
    Output
} from '@angular/core'

declare const tinymce: any

@Component({
    selector: 'text-editor',
    templateUrl: './texteditor.component.html',
    styleUrls: ['./texteditor.component.css']
})
export class TextEditorComponent implements AfterViewInit, OnDestroy {
    ClientSide: boolean
    TinyMCELoaded: boolean
    Editor: any
    Saved: boolean

    @Input() elementId: string
    @Input() text: string
    @Output() onSubmit = new EventEmitter<string>()
    @Output() onEditorContentChange = new EventEmitter<string>()

    constructor() {
        this.ClientSide = typeof window !== 'undefined'
        this.Saved = true
    }

    handleSubmit() {
        this.Saved = true
        const content = this.Editor.getContent()
        this.onSubmit.emit(content);
    }

    ngAfterViewInit() {
        if (this.ClientSide) {
            require.ensure([
                'tinymce'
            ], require => {
                require('tinymce')
                require('tinymce/themes/modern')
                require('tinymce/plugins/table')
                require('tinymce/plugins/link')
                require('tinymce/plugins/lists')
                require('tinymce/plugins/image')
                require('tinymce/plugins/imagetools')
                require('tinymce/plugins/save')
                require('tinymce/plugins/wordcount')

                this.TinyMCELoaded = true

                tinymce.init({
                    selector: '#' + this.elementId,
                    plugins: [ 'link', 'table', 'lists', 'image', 'imagetools', 'save', 'wordcount' ],
                    toolbar: 'undo redo | styleselect | bold italic | alignleft aligncenter alignright alignjustify | bullist numlist outdent indent | link image | save media | codesample help',
                    menubar: false,
                    save_onsavecallback: () => this.handleSubmit(),
                    skin_url: '/assets/tinymce/skins/lightgray', // Skins is in the wwwroot
                    setup: editor => {
                        this.Editor = editor
                        editor.on('keyup change undo redo', () => {
                            const content = editor.getContent()
                            this.Saved = false
                            this.onEditorContentChange.emit(content)
                        })
                    }
                })
            })
        }
    }

    ngOnDestroy() {
        if (this.ClientSide) {
            tinymce.remove(this.Editor)
        }
    }
}
```

<script src="https://gist.github.com/OskarKlintrot/8f3b684faeb93f8d2b0946f63b70bd61.js"></script>