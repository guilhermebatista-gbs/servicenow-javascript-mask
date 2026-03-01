# Input Masks for ServiceNow Service Catalog & Record Producers

# Input Masks for ServiceNow Service Catalog & Record Producers

Add real-time input formatting to ServiceNow Service Portal fields using [iMask.js](https://imask.js.org/) — a lightweight, dependency-free JavaScript masking library.

> **No native support?** ServiceNow doesn't provide built-in input masking for catalog variables. This guide shows you how to implement it using iMask.js with Catalog Client Scripts.

---

## Table of Contents

- [Why Input Masks?](#why-input-masks)
- [How It Works](#how-it-works)
- [Prerequisites](#prerequisites)
- [Setup Guide](#setup-guide)
  - [Step 1 — Add iMask.js to Your Service Portal Theme](#step-1--add-imaskjs-to-your-service-portal-theme)
  - [Step 2 — Create the Catalog Variable](#step-2--create-the-catalog-variable)
  - [Step 3 — Add the Catalog Client Script](#step-3--add-the-catalog-client-script)
- [Mask Examples](#mask-examples)
  - [Phone Number](#phone-number)
  - [Social Security Number (SSN)](#social-security-number-ssn)
  - [ZIP Code](#zip-code)
  - [Credit Card Number](#credit-card-number)
  - [Date (mm/dd/yyyy)](#date-mmddyyyy)
  - [Currency (USD)](#currency-usd)
  - [EIN (Employer Identification Number)](#ein-employer-identification-number)
  - [IP Address](#ip-address)
  - [Custom Alphanumeric](#custom-alphanumeric)
- [Advanced Configuration](#advanced-configuration)
  - [Lazy Mode (Placeholder Visibility)](#lazy-mode-placeholder-visibility)
  - [Placeholder Character](#placeholder-character)
  - [Overwrite Mode](#overwrite-mode)
  - [Dynamic Masks (Multiple Formats)](#dynamic-masks-multiple-formats)
  - [Getting Unmasked Values](#getting-unmasked-values)
- [Multiple Masks in One Script](#multiple-masks-in-one-script)
- [Applying to Record Producers](#applying-to-record-producers)
- [A Note on setTimeout](#a-note-on-settimeout)
- [Troubleshooting](#troubleshooting)
- [References](#references)

---

## Why Input Masks?

| Approach | User Experience | Data Quality | Accessibility |
|---|---|---|---|
| **No validation** | Users guess the format | Inconsistent data | No guidance |
| **Regex (onChange)** | Error message after typing | Good, but frustrating | Screen readers struggle with tooltip errors |
| **Input Mask** | Format guides the user in real time | Excellent — enforced as they type | Clear, predictable input behavior |

Input masks guide users while they type, eliminating formatting errors at the source. This means cleaner data, better UX, and fewer support tickets.

---

## How It Works

```
┌─────────────────────────────────────────────────┐
│  Service Portal Theme                           │
│  └── JS Include: iMask.js (CDN)                │
│                                                 │
│  Catalog Item / Record Producer                 │
│  └── Variable: Single Line Text                 │
│  └── Catalog Client Script (onLoad)             │
│       └── IMask(element, { mask options })       │
└─────────────────────────────────────────────────┘
```

The iMask.js library is loaded globally via the Service Portal theme. Then, a Catalog Client Script (type `onLoad`) targets the variable's DOM element and applies the mask configuration.

---

## Prerequisites

- ServiceNow instance (San Diego or later recommended)
- Access to **Service Portal Theme** configuration
- Access to create **Catalog Client Scripts**
- A **Catalog Item** or **Record Producer** with at least one Single Line Text variable

---

## Setup Guide

### Step 1 — Add iMask.js to Your Service Portal Theme

1. Navigate to **Service Portal > Portals** and open your portal.
2. Click on the **Theme** to open the theme record.
3. Scroll down to the **JS Includes** related list.
4. Click **New** and fill in:

| Field | Value |
|---|---|
| **Display name** | `iMask` |
| **Source** | `URL` |
| **JS file URL** | `https://unpkg.com/imask` |

5. Click **Submit**.

> **Note:** This makes iMask.js available on every page that uses this portal theme. The library is ~15KB gzipped.

---

### Step 2 — Create the Catalog Variable

1. Open your **Catalog Item** or **Record Producer**.
2. In the **Variables** related list, click **New**.
3. Configure:

| Field | Value |
|---|---|
| **Type** | `Single Line Text` |
| **Question** | e.g., `Phone Number` |
| **Name** | e.g., `phone_number` |

4. Click **Submit**.

> **Important:** Note the variable **Name** — you will reference it in the client script as `sp_formfield_<variable_name>`.

---

### Step 3 — Add the Catalog Client Script

1. On the same Catalog Item, go to the **Catalog Client Scripts** related list.
2. Click **New** and configure:

| Field | Value |
|---|---|
| **Name** | `onLoad Input Mask - Phone` |
| **Applies to** | `A Catalog Item` |
| **UI Type** | `All` |
| **Type** | `onLoad` |
| **Isolate script** | ✅ **Checked** (required!) |
| **Catalog Item** | *(your catalog item)* |

3. In the **Script** field, paste:

```javascript
function onLoad() {
    setTimeout(function() {
        var phoneMask = new IMask(top.document.getElementById('sp_formfield_phone_number'), {
            mask: '(000) 000-0000',
            lazy: false,
            placeholderChar: '_'
        });
    }, 100);
}
```

4. Click **Submit**.

> **Critical Settings:**
> - **Isolate script** must be **checked** — this allows access to the `top` document and the iMask library loaded in the portal theme.
> - The element ID follows the pattern `sp_formfield_<variable_name>`.
> - The `setTimeout` with a small delay (100ms) ensures the Service Portal has fully rendered the input element before the mask is applied.

---

## Mask Examples

Below are ready-to-use mask configurations. Replace the variable name in `getElementById` with your own.

### Phone Number

```javascript
function onLoad() {
    setTimeout(function() {
        var mask = new IMask(top.document.getElementById('sp_formfield_phone_number'), {
            mask: '(000) 000-0000',
            lazy: false,
            placeholderChar: '_'
        });
    }, 100);
}
```
**Output:** `(___) ___-____` → `(555) 123-4567`

---

### Social Security Number (SSN)

```javascript
function onLoad() {
    setTimeout(function() {
        var mask = new IMask(top.document.getElementById('sp_formfield_ssn'), {
            mask: '000-00-0000',
            lazy: false,
            placeholderChar: '_'
        });
    }, 100);
}
```
**Output:** `___-__-____` → `123-45-6789`

---

### ZIP Code

Standard 5-digit:

```javascript
function onLoad() {
    setTimeout(function() {
        var mask = new IMask(top.document.getElementById('sp_formfield_zip_code'), {
            mask: '00000',
            lazy: false,
            placeholderChar: '_'
        });
    }, 100);
}
```
**Output:** `_____` → `90210`

ZIP+4 format:

```javascript
function onLoad() {
    setTimeout(function() {
        var mask = new IMask(top.document.getElementById('sp_formfield_zip_code'), {
            mask: '00000-0000',
            lazy: false,
            placeholderChar: '_'
        });
    }, 100);
}
```
**Output:** `_____-____` → `90210-1234`

Dynamic (accepts both):

```javascript
function onLoad() {
    setTimeout(function() {
        var mask = new IMask(top.document.getElementById('sp_formfield_zip_code'), {
            mask: [
                { mask: '00000' },
                { mask: '00000-0000' }
            ]
        });
    }, 100);
}
```

---

### Credit Card Number

```javascript
function onLoad() {
    setTimeout(function() {
        var mask = new IMask(top.document.getElementById('sp_formfield_credit_card'), {
            mask: '0000 0000 0000 0000',
            lazy: false,
            placeholderChar: '_'
        });
    }, 100);
}
```
**Output:** `____ ____ ____ ____` → `4532 1234 5678 9012`

---

### Date (mm/dd/yyyy)

```javascript
function onLoad() {
    setTimeout(function() {
        var mask = new IMask(top.document.getElementById('sp_formfield_date'), {
            mask: Date,
            pattern: 'm{/}`d{/}`Y',
            lazy: false,
            blocks: {
                d: { mask: IMask.MaskedRange, from: 1, to: 31, maxLength: 2 },
                m: { mask: IMask.MaskedRange, from: 1, to: 12, maxLength: 2 },
                Y: { mask: IMask.MaskedRange, from: 1900, to: 2099 }
            },
            format: function(date) {
                var day = date.getDate();
                var month = date.getMonth() + 1;
                var year = date.getFullYear();
                if (day < 10) day = '0' + day;
                if (month < 10) month = '0' + month;
                return month + '/' + day + '/' + year;
            },
            parse: function(str) {
                var parts = str.split('/');
                return new Date(parts[2], parts[0] - 1, parts[1]);
            },
            autofix: true,
            overwrite: true
        });
    }, 100);
}
```
**Output:** `mm/dd/yyyy` → `12/25/2025`

---

### Currency (USD)

```javascript
function onLoad() {
    setTimeout(function() {
        var mask = new IMask(top.document.getElementById('sp_formfield_amount'), {
            mask: '$ num',
            blocks: {
                num: {
                    mask: Number,
                    thousandsSeparator: ',',
                    radix: '.',
                    scale: 2,
                    padFractionalZeros: true,
                    normalizeZeros: true,
                    min: 0
                }
            }
        });
    }, 100);
}
```
**Output:** `$ 1,234.56`

---

### EIN (Employer Identification Number)

```javascript
function onLoad() {
    setTimeout(function() {
        var mask = new IMask(top.document.getElementById('sp_formfield_ein'), {
            mask: '00-0000000',
            lazy: false,
            placeholderChar: '_'
        });
    }, 100);
}
```
**Output:** `__-_______` → `12-3456789`

---

### IP Address

```javascript
function onLoad() {
    setTimeout(function() {
        var mask = new IMask(top.document.getElementById('sp_formfield_ip_address'), {
            mask: 'num.num.num.num',
            blocks: {
                num: {
                    mask: IMask.MaskedRange,
                    from: 0,
                    to: 255
                }
            }
        });
    }, 100);
}
```
**Output:** `192.168.1.1`

---

### Custom Alphanumeric

For a license plate or serial number format (e.g., `ABC-1234`):

```javascript
function onLoad() {
    setTimeout(function() {
        var mask = new IMask(top.document.getElementById('sp_formfield_serial'), {
            mask: 'aaa-0000',
            definitions: {
                'a': /[A-Za-z]/,
                '0': /[0-9]/
            },
            prepareChar: function(str) { return str.toUpperCase(); },
            lazy: false,
            placeholderChar: '_'
        });
    }, 100);
}
```
**Output:** `___-____` → `ABC-1234`

---

## Advanced Configuration

### Lazy Mode (Placeholder Visibility)

```javascript
lazy: false   // Always show placeholder (e.g., "(___) ___-____")
lazy: true    // Only show mask as user types (default)
```

### Placeholder Character

```javascript
placeholderChar: '_'   // (___) ___-____
placeholderChar: '#'   // (###) ###-####
placeholderChar: ' '   // (   )    -
```

### Overwrite Mode

```javascript
overwrite: true     // Replaces characters instead of inserting
overwrite: 'shift'  // Shifts existing characters forward
```

### Dynamic Masks (Multiple Formats)

Useful when a field can accept different formats:

```javascript
function onLoad() {
    setTimeout(function() {
        var mask = new IMask(top.document.getElementById('sp_formfield_phone'), {
            mask: [
                { mask: '(000) 000-0000' },      // Standard US
                { mask: '+0 (000) 000-0000' }     // With country code
            ]
        });
    }, 100);
}
```

iMask automatically selects the best matching mask based on the input length.

### Getting Unmasked Values

If you need to retrieve the raw value (without formatting) for server-side processing, you can access `mask.unmaskedValue` in an onChange script:

```javascript
// In a separate onChange Catalog Client Script
function onChange(control, oldValue, newValue, isLoading) {
    if (isLoading) return;
    // The masked input stores the formatted value
    // e.g., "(555) 123-4567" → unmasked: "5551234567"
    // Access via the mask instance if needed
}
```

> **Tip:** ServiceNow stores the displayed (masked) value in the variable. If you need only digits, handle stripping on the server side with a Business Rule or Script Include.

---

## Multiple Masks in One Script

You can apply masks to multiple variables in a single Catalog Client Script:

```javascript
function onLoad() {
    setTimeout(function() {
        // Phone mask
        new IMask(top.document.getElementById('sp_formfield_phone_number'), {
            mask: '(000) 000-0000',
            lazy: false,
            placeholderChar: '_'
        });

        // SSN mask
        new IMask(top.document.getElementById('sp_formfield_ssn'), {
            mask: '000-00-0000',
            lazy: false,
            placeholderChar: '_'
        });

        // ZIP Code mask
        new IMask(top.document.getElementById('sp_formfield_zip_code'), {
            mask: '00000',
            lazy: false,
            placeholderChar: '_'
        });
    }, 100);
}
```

---

## Applying to Record Producers

The setup is identical to Catalog Items. The only differences:

1. The Catalog Client Script's **Applies to** field should be set to match your Record Producer.
2. The element ID still follows the pattern `sp_formfield_<variable_name>`.
3. Make sure your Record Producer is associated with a portal that uses the theme where iMask.js is included.

---

## A Note on setTimeout

This solution uses `setTimeout` to delay mask application until the DOM is ready. On the **client side** (Service Portal), `setTimeout` has always been available.

Interestingly, as [Nicola Attico](https://www.linkedin.com/in/nicola-attico/) (Sr Product Manager - Innovation @ ServiceNow) recently shared, `setTimeout()` now also works **server-side** in ServiceNow's newer JS engine (tested on ZP0 and YP6). For years, pausing server-side execution required `gs.sleep()`, which caused blocking and performance risks. The new non-blocking `setTimeout` is a welcome addition to the platform.

---

## Troubleshooting

| Issue | Solution |
|---|---|
| **Mask not appearing** | Ensure `Isolate script` is checked on the Catalog Client Script |
| **`IMask is not defined`** | Verify the JS Include is on the correct portal theme and the Source is `URL` |
| **Element not found** | Check the variable name matches `sp_formfield_<name>`. Try increasing the `setTimeout` delay to 200–500ms |
| **Mask works on portal but not in platform view** | iMask.js is loaded via the Service Portal theme — it only works on portal pages |
| **Multiple variables need masks** | Add multiple `IMask()` calls inside the same `setTimeout` block |
| **Value not saving correctly** | ServiceNow stores the masked value. Use a Before Insert Business Rule to strip formatting if needed |

### Debugging Tips

Open the browser console (F12) and test:

```javascript
// Check if iMask is loaded
typeof IMask  // Should return "function"

// Check if the element exists
document.getElementById('sp_formfield_phone_number')  // Should return an <input> element

// For Service Portal, try with top.document
top.document.getElementById('sp_formfield_phone_number')
```

---

## References

- [iMask.js Official Documentation](https://imask.js.org/)
- [iMask.js GitHub Repository](https://github.com/uNmAnNeR/imaskjs)
- [iMask.js on unpkg CDN](https://unpkg.com/imask)
- [ServiceNow Community — Service Catalog Input Masking](https://community.servicenow.com/community?id=community_article&sys_id=c8080ce5db334990b5d6e6be1396195b)
- [ServiceNow Docs — Catalog Client Scripts](https://docs.servicenow.com)

---

## License

This documentation is provided as-is for educational purposes. iMask.js is licensed under the [MIT License](https://github.com/uNmAnNeR/imaskjs/blob/master/LICENSE).

---

**Author:** Guilherme Batista da Silva  
**ServiceNow Developer** | ServiceNow Rising Star 2024 & 2025 | ITSM | CSM | App Engine
