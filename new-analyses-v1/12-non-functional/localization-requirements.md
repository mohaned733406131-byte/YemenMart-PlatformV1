# Localization Requirements - YemenMart

## Document Information
| Property | Value |
|----------|-------|
| Document ID | YEM-NFR-LOC-001 |
| Version | 1.0 |
| Status | Active |
| Last Updated | 2026-09-12 |

---

## 1. Language Support

### 1.1 Primary Languages
| Language | Code | Direction | Priority | Status |
|----------|------|-----------|----------|--------|
| Arabic | ar | RTL | Primary | Active |
| English | en | LTR | Secondary | Active |

### 1.2 Language Toggle
| Feature | Implementation |
|---------|---------------|
| Toggle Location | Header (top-right) |
| Default Language | Arabic (ar) |
| Persistence | User preference in DB + localStorage |
| Auto-detection | Browser language header |
| URL Support | /ar/ and /en/ prefixes |

---

## 2. RTL/LTR Layout

### 2.1 Layout Mirroring
| Component | RTL Behavior |
|-----------|-------------|
| Navigation | Mirrored (right-to-left) |
| Sidebar | Right side |
| Forms | Labels right, inputs left |
| Buttons | Icon right, text left |
| Tables | Columns mirrored |
| Modal/Dialog | Close button left |
| Breadcrumbs | Reversed order |
| Pagination | Reversed order |
| Progress Bars | Right-to-left fill |
| Carousels | Right-to-left scroll |

### 2.2 CSS Strategy
| Approach | Implementation |
|----------|---------------|
| Direction Property | dir="rtl" on html element |
| Logical Properties | margin-inline-start, padding-inline-end |
| CSS Layers | RTL overrides in separate layer |
| Framework | Tailwind CSS with RTL plugin |

### 2.3 Text Alignment
| Element | RTL Alignment | LTR Alignment |
|---------|--------------|---------------|
| Headings | Right | Left |
| Paragraphs | Right | Left |
| Lists | Right | Left |
| Form Labels | Right | Left |
| Navigation Items | Right | Left |
| Tables | Right | Left |
| Code Blocks | Left (no mirror) | Left |

---

## 3. Arabic Text Rendering

### 3.1 Font Configuration
| Font Type | Font Family | Fallbacks |
|-----------|-------------|-----------|
| Arabic Body | Noto Sans Arabic | Arial, sans-serif |
| Arabic Heading | Cairo | Noto Sans Arabic, sans-serif |
| English Body | Inter | Noto Sans Arabic, sans-serif |
| English Heading | Inter | Arial, sans-serif |
| Monospace | JetBrains Mono | Courier New, monospace |

### 3.2 Typography Rules
| Rule | Implementation |
|------|---------------|
| Line Height | 1.8 for Arabic, 1.5 for English |
| Letter Spacing | 0.02em for Arabic, normal for English |
| Word Spacing | 0.05em for Arabic, normal for English |
| Font Size | 16px base for both languages |
| Minimum Font Size | 12px for readability |

### 3.3 Text Expansion
| Element | English | Arabic | Expansion Factor |
|---------|---------|--------|-----------------|
| Buttons | 10 chars | 15 chars | 1.5x |
| Labels | 15 chars | 25 chars | 1.7x |
| Headings | 20 chars | 35 chars | 1.75x |
| Error Messages | 50 chars | 80 chars | 1.6x |
| Tooltips | 30 chars | 50 chars | 1.67x |
| Navigation | 10 chars | 18 chars | 1.8x |

---

## 4. Currency Formatting

### 4.1 Supported Currencies
| Currency | Code | Symbol | Position | Decimals |
|----------|------|--------|----------|----------|
| Saudi Riyal | SAR | ر.س | After amount | 2 |
| Yemeni Rial | YER | ي.ر | After amount | 0 |
| US Dollar | USD | $ | Before amount | 2 |

### 4.2 Currency Display
| Language | SAR Display | Example |
|----------|------------|---------|
| Arabic | ر.س ١٢٣٫٤٥ | ر.س ١٢٣٫٤٥ |
| English | SAR 123.45 | SAR 123.45 |

### 4.3 Number Systems
| System | Digits | Usage |
|--------|--------|-------|
| Western Arabic | 0123456789 | English UI |
| Eastern Arabic | ٠١٢٣٤٥٦٧٨٩ | Arabic UI |
| User Preference | Configurable | Settings |

### 4.4 Currency Formatting Rules
| Rule | Arabic | English |
|------|--------|---------|
| Thousand Separator | ٬ (Arabic comma) | , (comma) |
| Decimal Separator | ٫ (Arabic decimal) | . (period) |
| Negative Amount | -١٢٣٫٤٥ | -123.45 |
| Positive Amount | ١٢٣٫٤٥ | 123.45 |
| Currency Symbol | After number | After code |

---

## 5. Date and Time Formatting

### 5.1 Date Formats
| Format Type | Arabic | English | Example |
|------------|--------|---------|---------|
| Short | dd/mm/yyyy | mm/dd/yyyy | ١٢/٠٩/٢٠٢٦ |
| Long | dd MMMM yyyy | MMMM dd, yyyy | ١٢ سبتمبر ٢٠٢٦ |
| ISO | yyyy-mm-dd | yyyy-mm-dd | 2026-09-12 |

### 5.2 Time Formats
| Format Type | Arabic | English | Example |
|------------|--------|---------|---------|
| 12-hour | hh:mm ص/م | hh:mm AM/PM | ٠٢:٣٠ م |
| 24-hour | HH:mm | HH:mm | 14:30 |

### 5.3 Calendar System
| Calendar | Usage | Implementation |
|----------|-------|---------------|
| Gregorian | Primary | date-fns / dayjs |
| Hijri | Alternative | hijri-converter library |

### 5.4 Day and Month Names
| Element | Arabic | English |
|---------|--------|---------|
| Sunday | الأحد | Sunday |
| Monday | الاثنين | Monday |
| Tuesday | الثلاثاء | Tuesday |
| Wednesday | الأربعاء | Wednesday |
| Thursday | الخميس | Thursday |
| Friday | الجمعة | Friday |
| Saturday | السبت | Saturday |
| January | يناير | January |
| February | فبراير | February |
| March | مارس | March |
| April | أبريل | April |
| May | مايو | May |
| June | يونيو | June |
| July | يوليو | July |
| August | أغسطس | August |
| September | سبتمبر | September |
| October | أكتوبر | October |
| November | نوفمبر | November |
| December | ديسمبر | December |

---

## 6. Address Localization

### 6.1 Address Format
| Field | Arabic Label | English Label | Required |
|-------|-------------|---------------|----------|
| Full Name | الاسم الكامل | Full Name | Yes |
| Building Number | رقم المبنى | Building Number | Yes |
| Street Name | اسم الشارع | Street Name | Yes |
| District | الحي | District | Yes |
| City | المدينة | City | Yes |
| Region | المنطقة | Region | Yes |
| Postal Code | الرمز البريدي | Postal Code | Yes |
| Country | الدولة | Country | Yes |
| Phone | الهاتف | Phone | Yes |

### 6.2 Saudi Address Standards (SASO)
| Requirement | Implementation |
|------------|----------------|
| Building Number | 4-digit number |
| Street Name | Arabic + English |
| District | Arabic + English |
| City | Predefined list |
| Region | 13 regions of Saudi Arabia |
| Postal Code | 5 digits |

---

## 7. Number Formatting

### 7.1 Number Display
| Type | Arabic | English |
|------|--------|---------|
| Integer | ١٬٢٣٤ | 1,234 |
| Decimal | ١٬٢٣٤٫٥٦ | 1,234.56 |
| Percentage | ١٥٪ | 15% |
| Phone | ٠٥٥٥١٢٣٤٥٦ | 0555123456 |
| Order Number | #١٢٣٤٥ | #12345 |

### 7.2 Number Input
| Feature | Implementation |
|---------|---------------|
| Accept Both Systems | Parse both 0-9 and ٠-٩ |
| Display | Based on language preference |
| Validation | Accept both formats |
| Conversion | Auto-convert on input |

---

## 8. Right-to-Left (RTL) Components

### 8.1 Component-Specific RTL
| Component | RTL Behavior |
|-----------|-------------|
| Text Input | Text aligns right, cursor right |
| Select/Dropdown | Options right-aligned |
| Checkbox/Radio | Icon on right, label on left |
| Slider | Right-to-left movement |
| Progress Bar | Fills right-to-left |
| Carousel | Navigates right-to-left |
| Tabs | First tab on right |
| Accordion | Chevron on left |
| Toast/Notification | Appears from right |
| Modal | Close button on left |
| Tooltip | Position mirrored |

### 8.2 Icon Mirroring
| Icon Type | Mirror in RTL |
|-----------|--------------|
| Navigation arrows | Yes |
| Back/Forward | Yes |
| Chevron/Arrow | Yes |
| Progress indicators | Yes |
| Loading spinners | No |
| Play/Pause | No |
| Checkmarks | No |
| Close (X) | No |

---

## 9. Content Management

### 9.1 Translation Management
| Aspect | Tool/Approach |
|--------|--------------|
| Translation Tool | Crowdin / Lokalise |
| Translation Memory | Centralized TM |
| Glossary | Industry-specific terms |
| QA Checks | Automated + manual |
| Review Process | Two-pass review |

### 9.2 Content Structure
| Content Type | Bilingual Required | Default |
|-------------|-------------------|---------|
| Product Names | Yes | Arabic |
| Product Descriptions | Yes | Arabic |
| Category Names | Yes | Arabic |
| UI Labels | Yes | Arabic |
| Error Messages | Yes | Arabic |
| Marketing Content | Yes | Arabic |
| Legal Terms | Yes | Arabic |

### 9.3 Translation Quality
| Metric | Target |
|--------|--------|
| Accuracy | > 98% |
| Completeness | 100% |
| Consistency | > 95% |
| Fluency | > 95% |
| Terminology | 100% compliant |

---

## 10. Search Localization

### 10.1 Search Features
| Feature | Implementation |
|---------|---------------|
| Arabic Search | Full-text search in Arabic |
| English Search | Full-text search in English |
| Mixed Search | Support both languages |
| Transliteration | Arabic to English mapping |
| Fuzzy Matching | Handle typos and variations |
| Synonyms | Arabic and English synonyms |

### 10.2 Search Tokens
| Arabic | English | Mapping |
|--------|---------|---------|
| هاتف | phone | Synonym |
| جوال | mobile | Synonym |
| حاسوب | computer | Synonym |
| تلفاز | television | Synonym |

---

## 11. Notification Localization

### 11.1 Notification Channels
| Channel | Arabic | English |
|---------|--------|---------|
| Email | Full support | Full support |
| SMS | Full support | Full support |
| Push Notification | Full support | Full support |
| In-App | Full support | Full support |
| WhatsApp | Full support | Full support |

### 11.2 Notification Content
| Type | Arabic Template | English Template |
|------|----------------|------------------|
| Order Confirmation | تم تأكيد طلبك #{{orderNumber}} | Your order #{{orderNumber}} is confirmed |
| Shipping | تم شحن طلبك #{{orderNumber}} | Your order #{{orderNumber}} has shipped |
| Delivery | تم توصيل طلبك #{{orderNumber}} | Your order #{{orderNumber}} has been delivered |
| Payment | تم استلام دفعتك | Payment received |

---

## 12. Testing Requirements

### 12.1 Localization Testing
| Test Type | Frequency | Scope |
|-----------|-----------|-------|
| Visual Regression | Per release | All pages |
| Text Overflow | Per release | All components |
| RTL Layout | Per release | All pages |
| Number Formatting | Per release | All numeric displays |
| Date Formatting | Per release | All date displays |
| Currency Formatting | Per release | All price displays |

### 12.2 Test Checklist
| Item | Arabic | English |
|------|--------|---------|
| Text alignment | Right-aligned | Left-aligned |
| Text overflow | No truncation | No truncation |
| Number display | Eastern Arabic | Western Arabic |
| Date display | dd/mm/yyyy | mm/dd/yyyy |
| Currency display | SAR after number | SAR before amount |
| Form validation | Arabic messages | English messages |
| Error messages | Arabic | English |
| Success messages | Arabic | English |

### 12.3 Localization Metrics
| Metric | Target |
|--------|--------|
| Translation Coverage | 100% |
| Missing Translations | 0 |
| Inconsistent Translations | 0 |
| RTL Layout Issues | 0 |
| Formatting Issues | 0 |
| Cultural Appropriateness | 100% |
