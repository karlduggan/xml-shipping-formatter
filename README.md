# PO XML Builder - Overseas Shipping

A Vue 3 + Vite application for generating ASN (Advanced Shipment Notification) XML files for overseas shipments.

## Features

✨ **Simple Form-Based Interface**
- Fill in shipment header information once
- Add vessel/shipping details
- Dynamically add/remove line items
- Real-time XML generation

📦 **Line Items Management**
- Add unlimited line items
- Each item has: Product Number, Ordered Quantity, Actual Quantity
- Container info applies to all items
- Auto-incrementing Line Item Numbers

💾 **Export Options**
- Download XML file directly
- Copy XML to clipboard
- Preview formatted XML output

🎨 **User-Friendly Design**
- Modern, responsive UI
- Form validation
- Toast notifications
- Mobile-friendly

## Quick Start

### Prerequisites
- Node.js 16+ and npm

### Installation

```bash
# Install dependencies
npm install
```

### Development

```bash
# Start development server (runs on http://localhost:5173)
npm run dev
```

### Build for Production

```bash
# Create optimized build
npm run build

# Preview production build
npm run preview
```

## How to Use

1. **Fill Header Info**: Enter Sender ID, Receiver ID, Message Date, etc.
2. **Add Vessel Details**: Input vessel name, ports, dates, container info
3. **Add Line Items**: Click "Add Line Item" to add products
4. **Generate XML**: Click "Generate XML" button
5. **Download/Copy**: Download the XML file or copy to clipboard

## Form Fields

### Header Section
- Sender ID
- Receiver ID
- Message Date & Time
- Envelope ID
- Purchase Order Number

### Vessel & Shipping
- Status Code & Description
- Vessel Name
- ETS (Expected Time of Sailing)
- ETA (Expected Time of Arrival)
- POL (Port of Loading)
- POD (Port of Discharge)
- Container Number
- Container Type (20DC, 40DC, 40HC, 20HC)

### Line Items
- Product Number
- Ordered Quantity
- Actual Quantity
- Line Item Number (auto-generated)

## XML Output Structure

The generated XML follows this structure:

```xml
<?xml version="1.0" encoding="utf-8"?>
<ASN>
  <Envelope>
    <!-- Header info -->
  </Envelope>
  <Event>
    <Status><!-- Status info --></Status>
    <Vessel><!-- Vessel info --></Vessel>
    <Details><!-- Line item 1 --></Details>
    <Details><!-- Line item 2 --></Details>
    <!-- More line items... -->
  </Event>
</ASN>
```

## Tips

- Container Number and Type apply to all line items in the shipment
- Line Item Numbers are auto-incremented (1, 2, 3, etc.)
- Date format is automatically converted to DD-MM-YYYY for XML output
- All fields are validated before XML generation

## Development

The app is built with:
- **Vue 3** - Progressive JavaScript framework
- **Vite** - Next generation frontend build tool
- **ES Modules** - Modern JavaScript standard

## License

MIT
# xml-shipping-formatter
