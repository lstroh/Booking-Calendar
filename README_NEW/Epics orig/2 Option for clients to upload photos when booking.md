Got it ✅ — let’s break down the **“Option for clients to upload photos when booking”** into clear, testable requirements and success criteria you can later use as a checklist when reviewing the plugin implementation.

---

## **Requirements Breakdown**

### 1. Upload Functionality

* [ ] Clients can upload photos directly in the booking form.
* [ ] Support for **multiple file uploads** (not just one).
* [ ] Allowed file formats configurable (e.g., JPG, PNG, HEIC, PDF if needed).
* [ ] File size limit configurable (e.g., 10MB per file, total 50MB).


### 2. Storage & Access

* [ ] Photos are stored securely in WordPress (e.g., media library or dedicated folder).
* [ ] Files are linked to the specific booking record in the database.
* [ ] Tradesperson (admin or assigned staff) can view/download photos from the booking details screen.
* [ ] Access permissions respected — only tradesperson/admin, not other clients.



### 3. Consent & Compliance

* [ ] Clear **consent checkbox** before upload (e.g., “I consent to my photos being stored and used for service purposes”).
* [ ] Checkbox must be required before upload is submitted.
* [ ] Consent record stored with booking data (timestamp + wording).
* [ ] GDPR-compliant — clients can request deletion of their photos later.



### 4. User Experience

* [ ] Drag-and-drop upload support.
* [ ] Preview thumbnails before final submission.
* [ ] Option to remove/edit uploads before confirming booking.
* [ ] Mobile-friendly upload (camera integration on phones).

### 5. Notifications & Integration

* [ ] Tradesperson notified (email/SMS) when booking includes photos.
* [ ] Photos included as attachment or secure link in email notifications (optional, configurable).
* [ ] API/Webhook ready if integration with other tools (e.g., CRM, quoting).



## **Success Criteria**

✅ Clients can upload multiple photos during booking.
✅ Only supported formats & sizes are accepted (error messages shown if invalid).
✅ Uploads are tied to the correct booking in WordPress.
✅ Consent checkbox must be ticked before form submission is allowed.
✅ Admin/tradesperson can view/download photos from booking details.
✅ GDPR compliance: consent stored + ability to delete photos on request.
✅ Photos display previews in booking form before submission.
✅ Works smoothly on desktop and mobile.
✅ No other clients can access the uploaded files.
✅ Notifications (email/SMS) correctly include photo references or links.

---

Would you like me to also **estimate development effort & dependencies** (e.g., media handling, GDPR logging, mobile support) so you can plan this feature against the others in your backlog?
