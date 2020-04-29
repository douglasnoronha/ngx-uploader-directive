# ng-file-uploader
---

Angular 9 File Uploader.
This project was generated with [Angular CLI](https://github.com/angular/angular-cli) version 7.3.8.

## Installation
---

Add `ng-file-uploader` module as dependency to your project.

```console
npm install ng-file-uploader --save
```

## Usage
---

1. Import `NgFileUploaderModule` into your AppModule or in module where you will use it.

```js
// app.module.ts
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { NgFileUploaderModule } from 'ng-file-uploader';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    NgFileUploaderModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

**or** Import `NgFileUploaderModule` into your SharedModule. This could be usefull if your project has nested Modules.

```js
// shared.module.ts
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { NgFileUploaderModule } from 'ng-file-uploader';

@NgModule({  
  imports: [
    BrowserModule,
    AppRoutingModule,
    NgFileUploaderModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class SharedModule { }
```

2. Data structures of Input events and upload output events of files.

```js

/**
 * File Upload Options.
 */
export interface IUploadOptions {
    requestConcurrency: number; // Number of request can be made at a time.
    maxFilesToAddInSingleRequest: number; // Number of files uploaded in single.
    allowedFileTypes?: Array<string>; // Allowed file content types.
    maxFileUploads?: number; // Max number of files that user can upload
    maxFileSize?: number; // Max size of the file in bytes that user can upload.
    logs?: boolean; // Flag to show the library logs. Default false
}

/**
 * Selected File Object.
 */
export interface ISelectedFile {
    id: string; // Unique id of selected file. generated by library.
    fileIndex: number; // file index of selected files.
    name: string; // Name of file.
    type: string; // Type of file.
    progress?: IUploadProgress; // File upload Progress.
    nativeFile?: File; // Native File.
    formData?: FormData; // Form data to upload with file.
    response?: any; // Response for the selected file.
}

/**
 * File Upload Progress.
 */
export interface IUploadProgress {
    status: 'Queue' | 'Uploading' | 'Done' | 'Cancelled'; // Progress stauts.
    data?: {
        percentage: number; // Progress percentage.
        speed: number; // Progress speed.
        speedHuman: string; // Progress spped human.
        startTime: number | null; // Progress start time.
        endTime: number | null; // Progress end time.
        eta: number | null; // Progress eta.
        etaHuman: string | null; // Progress eta human.
    }; // Upload progress data.
}

/**
 * Upload Input events that can be emit to ng-file-uploader.
 */
export interface IUploadInput {
    type: 'uploadAll' | 'uploadFile' | 'cancel' | 'cancelAll' | 'remove' | 'removeAll'; // Input event type.
    url?: string; // Input url.
    method?: string; // Input method.
    id?: string; // Input id of file to upload.
    fieldName?: string; // Input field name.
    fileIndex?: number; // Input file index to upload.
    file?: ISelectedFile; // Input selected file.
    formData?: FormData; // Input form data to pass with file.
    headers?: { [key: string]: string }; // Input headers to pass with upload request.
    withHeaders?: boolean; // Input upload with credentials.
}

/**
 * File Upload Output Events that emitted by ng-file-uploader.
 */
export interface IUploadOutput {
    type: 'init' | 'addedToQueue' | 'allAddedToQueue' | 'uploading' | 'done' | 'start' | 'cancelled' | 'dragOver' | 'dragOut' | 'drop' | 'removed' | 'removedAll' | 'rejected' | 'error'; // Output events.
    file?: ISelectedFile; // selected file.
    progress?: IUploadProgress; // Progress
    response?: any; // File upload api response.
}

```

## Example
---
**You can always run this working example by cloning this repository and build and run with command in terminal `npm start`.**

**Component code**

```js
// tslint:disable: max-line-length
import { Component, EventEmitter } from '@angular/core';
import { IUploadOptions, ISelectedFile, IUploadInput, IUploadOutput } from 'ng-file-uploader';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss']
})

export class AppComponent {
  title = 'ng-file-uploader';
  options: IUploadOptions;
  formData: FormData;
  files: Array<ISelectedFile>;
  uploadInput: EventEmitter<IUploadInput>;
  dragOver: boolean;
  uploadUrl = 'http://192.168.0.224:8099/api/blocklists/uploadblockednumberfile';


  /**
   * Default Constructor
   */
  constructor() {
    this.options = { requestConcurrency: 1, maxFilesToAddInSingleRequest: 10, maxFileUploads: 5, maxFileSize: 1000000, logs: true };
    this.files = new Array<ISelectedFile>();
    this.uploadInput = new EventEmitter<IUploadInput>();
    this.formData = new FormData();
  }

  onUploadOutput(output: IUploadOutput): void {
    console.log(output);
    switch (output.type) {
      case 'init':
        this.files = new Array<ISelectedFile>();
        break;
      case 'allAddedToQueue':
        // uncomment this if you want to auto upload files when added
        // startUpload();
        break;
      case 'addedToQueue':
        if (typeof output.file !== 'undefined') {
          this.files.push(output.file);
          console.log(this.files);
        }
        break;
      case 'uploading':
        if (typeof output.file !== 'undefined') {
          // update current data in files array for uploading file
          const index = this.files.findIndex((file) => typeof output.file !== 'undefined' && file.id === output.file.id);
          this.files[index] = output.file;
        }
        break;
      case 'removed':
        // remove file from array when removed
        this.files = this.files.filter((file: ISelectedFile) => file !== output.file);
        break;
      case 'dragOver':
        this.dragOver = true;
        break;
      case 'dragOut':
      case 'drop':
        this.dragOver = false;
        break;
      case 'done':
        // The file is downloaded
        break;
    }
  }

  startUpload(): void {
    this.formData.append('fileHasHeader', 'false');
    this.formData.append('delimiter', ',');

    const event: IUploadInput = {
      type: 'uploadAll',
      url: this.uploadUrl,
      method: 'POST',
      formData: this.formData
    };

    this.uploadInput.emit(event);
  }

  cancelUpload(id: string): void {
    this.uploadInput.emit({ type: 'cancel', id: id });
  }

  removeFile(id: string): void {
    this.uploadInput.emit({ type: 'remove', id: id });
  }

  removeAllFiles(): void {
    this.uploadInput.emit({ type: 'removeAll' });
  }
}
```

**Html code**

```html
<div class="drop-container" ngFileDrop [options]="options" (uploadOutput)="onUploadOutput($event)"
  [uploadInput]="uploadInput" [ngClass]="{ 'is-drop-over': dragOver }">
  <h1>Drag &amp; Drop</h1>
</div>

<label class="upload-button">
  <input type="file" ngFileSelect [options]="options" (uploadOutput)="onUploadOutput($event)"
    [uploadInput]="uploadInput" multiple>
  or choose file(s)
</label>

<button type="button" class="start-upload-btn" (click)="startUpload()">
  Start Upload
</button>
```

## Running demo on local machine
---

```console
npm start
```

- If you face any problem in running demo do `npm install` for libraray and then try with `npm start`.

### LICENCE

[MIT](https://github.com/jayprajapati857/ng-file-uploader/blob/master/LICENSE)