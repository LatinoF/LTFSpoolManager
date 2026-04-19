# LTF Spool Manager

A professional, high-performance web-based dashboard designed to track and manage 3D printing filament inventory. **LTF Spool Manager** provides real-time tracking of spool weights, material types, and print history with a sleek, dark-themed interface.

## 🚀 Key Features

- **Inventory Dashboard**: High-level overview of all spools with visual progress bars and color indicators.
- **Detailed Tracking**: Monitor remaining weight, brand, and material type (PLA, PETG, TPU, etc.).
- **Print History**: Log every print job with descriptions and dates to automatically deduct weight from the spool.
- **Advanced Color Picker**: Custom color selection for every spool to match your real-world filaments.
- **Data Portability**: Full support for Importing and Exporting your database as JSON files.
- **Responsive Design**: Optimized for both desktop and mobile use with touch-friendly controls and smooth transitions.
- **Zero Dependencies**: Pure HTML/JS/CSS logic using LocalStorage for data persistence—no backend required.

## 🛠 Technical Stack

- **Frontend**: Vanilla HTML5, CSS3 (Custom Variables, CSS Transitions).
- **Icons**: [Lucide Icons](https://lucide.dev/) for a clean, modern UI.
- **Data**: Browser `LocalStorage` for persistent storage.
- **State Management**: Custom JavaScript state handling for view switching and modal management.

## 📦 Installation & Usage

1. **Download**: Clone the repository or download the `index.html` file.
2. **Run**: Simply open `index.html` in any modern web browser.
3. **Add Spool**: Click the green **+** button to register your first filament spool.
4. **Log Print**: Click on a spool card, then click the **+** button inside the detail view to log a print job.
5. **Backup**: Use the settings menu (gear icon) to export your database regularly.

## 🎨 Visual Identity

- **Theme**: Deep Dark Blue / Slate palette.
- **Accents**: 
  - Blue (`#3b82f6`) for actions.
  - Green (`#10b981`) for successes/additions.
  - Red (`#ef4444`) for deletions/warnings.

## 📄 License

This project is open-source. Feel free to modify and use it for your personal 3D printing workflow.

---
*Created for 3D printing enthusiasts who value precision and organization.*
