# Rust File Decompressor

## What does the app do?
The program takes as an argument the name of the compressed file, then decompress it and prints the number of possibly corrupted files encountered.

## Programming Languages:
- Rust

## Frameworks & Libraries:
- zip - https://crates.io/crates/zip | https://github.com/zip-rs/zip

## Implementation:
First, we check if the user passed the input file path:
```rust
let program_arguments = std::env::args().collect::<Vec<String>>();

// check program arguments if they are correctly passed
if 2 != program_arguments.len() {
    eprintln!("Usage: {} <compressed-file>", program_arguments[0]);
    return;
}
```
One of the next important steps is to create the *ZIP* archive:
```rust
// creating the zip archive using zip library
let mut zip_archive = match zip::ZipArchive::new(input_file) {
    Ok(archive) => archive,
    Err(err) => {
        eprintln!("Cannot create the zip archive because of an error: {}.", err);
        return;
    }
};
```
After that, we iterate over the files within our archive and another important step is to check if the file is a directory:
```rust
// check if the file is a directory and create everything that is necessary, like parent directories & so on
if (*zip_file.name()).ends_with('/') {
    println!("File #{} extracted to \"{}\".", file_number, output_file_path_buffer.display());
    if let Err(error) = std::fs::create_dir_all(&output_file_path_buffer) {
        eprintln!("Cannot create the directory \"{}\" because of an error: {}.", output_file_path_buffer.display(), error);
    }
}
```
The last step is to set the permissions if the program runs on a **Unix** system:
```rust
// set the neccessary permissions if we're on a unix machine for user to have access to the files
#[cfg(unix)]
{
    use std::os::unix::fs::PermissionsExt;

    if let Some(mode) = zip_file.unix_mode() {
        if let Err(error) = std::fs::set_permissions(&output_file_path_buffer, std::fs::Permissions::from_mode(mode)) {
            eprintln!("Cannot set the permissions of the output file \"{}\" because of an error: {}.", output_file_path_buffer.display(), error);
        }
    }
}
```
For this step to be done, the creation of a block with the *#[cfg(unix)]* annotation will be needed as we can see in the code sample from above.