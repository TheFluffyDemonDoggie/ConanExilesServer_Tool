import tkinter as tk
from tkinter import messagebox, filedialog
import requests
from bs4 import BeautifulSoup
import os

def is_valid_url(url):
    return url.startswith("https://steamcommunity.com/sharedfiles/filedetails/")

def extract_mod_ids_in_order(collection_url):
    try:
        status_label.config(text="Fetching data... Please wait.")
        root.update_idletasks()  # Update the UI during long-running tasks
        
        # Fetch the collection page
        response = requests.get(collection_url)
        response.raise_for_status()  # Check for HTTP errors
        
        # Parse the page content
        soup = BeautifulSoup(response.text, 'html.parser')
        
        # Find mod links in the order they appear
        mod_links = soup.find_all('a', href=True)
        mod_ids = []
        
        for link in mod_links:
            href = link['href']
            if "sharedfiles/filedetails/?id=" in href:
                # Extract the part after "id="
                mod_id = href.split('id=')[1].split('&')[0]  # Get the number part only
                mod_ids.append(mod_id)
        
        # Keep the IDs in the original order
        return mod_ids
    
    except requests.exceptions.RequestException as e:
        messagebox.showerror("Error", f"Failed to fetch the collection: {e}")
        return []
    finally:
        status_label.config(text="")  # Reset the status label

def process_collection():
    # Get the URL from the input field
    collection_url = url_entry.get().strip()
    if not collection_url or not is_valid_url(collection_url):
        messagebox.showwarning("Warning", "Please enter a valid Steam collection URL!")
        return

    # Extract mod IDs
    mod_ids = extract_mod_ids_in_order(collection_url)
    if mod_ids:
        result_text.delete(1.0, tk.END)  # Clear previous results
        result_text.insert(tk.END, ",".join(mod_ids))  # Display IDs separated by commas
        messagebox.showinfo("Success", f"Found {len(mod_ids)} mod IDs in the correct order!")
    else:
        messagebox.showwarning("Warning", "No mod IDs found!")

def save_to_file():
    # Get the extracted mod IDs
    mod_ids = result_text.get(1.0, tk.END).strip()
    if not mod_ids:
        messagebox.showwarning("Warning", "No mod IDs to save!")
        return

    # Open file save dialog
    file_path = filedialog.asksaveasfilename(
        defaultextension=".txt",
        filetypes=[("Text files", "*.txt")],
        title="Save Mod IDs"
    )
    if not file_path:
        return  # User canceled the save dialog

    # Save to the selected file
    try:
        with open(file_path, "w") as file:
            file.write(mod_ids)  # Save the comma-separated IDs
        messagebox.showinfo("Saved", f"Mod IDs saved to {file_path}!")
    except Exception as e:
        messagebox.showerror("Error", f"Failed to save mod IDs: {e}")

root = tk.Tk()
root.title("Steam Collection Mod ID Extractor")

frame = tk.Frame(root)
frame.pack(pady=10)
url_label = tk.Label(frame, text="Steam Collection URL:")
url_label.pack(side=tk.LEFT, padx=5)
url_entry = tk.Entry(frame, width=50)
url_entry.pack(side=tk.LEFT, padx=5)
process_button = tk.Button(frame, text="Extract Mod IDs", command=process_collection)
process_button.pack(side=tk.LEFT, padx=5)

status_label = tk.Label(root, text="", fg="blue")
status_label.pack(pady=5)

result_text = tk.Text(root, width=80, height=20)
result_text.pack(pady=10)

save_button = tk.Button(root, text="Save to File", command=save_to_file)
save_button.pack(pady=5)

root.mainloop()
