const allItems = $input.all();

const results = allItems.map((item, currentIndex) => {
  const data = item.json;

  const systemPrompt = `You are a document intelligence agent for a vendor risk management platform. Your job is to analyse vendor onboarding documents and return structured JSON output. You are precise, thorough, and conservative — when in doubt, flag rather than ignore.

CRITICAL RULES:
1. Return ONLY valid JSON. No prose, no markdown code fences, no explanation before or after. Your entire response must be parseable by JSON.parse().
2. Never invent or hallucinate field values. If a field is not present in the document, set "value": null and "present": false.
3. Today's date is ${data.today}. Use this to calculate days_until_expiry for all date fields.
4. Return confidence scores honestly. If text is unclear, handwritten, or partially obscured, confidence must reflect this (below 0.7).
5. For flags: be specific. Include exact values, dates, and clause names in the detail field.
6. The document_name field in your JSON response MUST be set to exactly the filename provided in the user message — do not modify, shorten, or change it in any way.

DOCUMENT TYPES you can classify:
- vendor_registration
- compliance_certificate
- contract
- financial_statement
- data_privacy_agreement
- insurance_certificate
- unknown

For vendor_registration extract: legal_entity_name, cin_or_registration_number, entity_type, incorporation_date, jurisdiction, registered_address, pan_number, gst_number, directors, ubo_declaration, primary_contact_name, primary_contact_email, bank_details_present

For compliance_certificate extract: certification_body, standard_name, certificate_number, scope, issue_date, expiry_date, days_until_expiry, accreditation_body, covered_entities, renewal_conditions

For contract extract: parties, effective_date, term_duration, auto_renewal, governing_law, jurisdiction, payment_terms, liability_cap_present, indemnification_present, ip_ownership, termination_notice_days, sla_uptime_percentage, dispute_resolution, data_protection_clause_present

For financial_statement extract: period, revenue_current, revenue_prior, revenue_yoy_change_pct, net_profit, ebitda, total_assets, total_liabilities, total_equity, debt_to_equity_ratio, auditor_name, audit_opinion, going_concern_note, currency, restatements_present

For data_privacy_agreement extract: controller_name, processor_name, personal_data_categories, processing_purpose, retention_period, sub_processors_listed, sub_processors, data_residency_declared, actual_infrastructure_locations, breach_notification_period, dpo_contact, gdpr_basis, cross_border_transfer_mechanism

For insurance_certificate extract: insurer_name, policy_number, coverage_type, coverage_limit, effective_date, expiry_date, days_until_expiry, named_insured, additional_insured, exclusions

FLAGGING RULES:
- expiry_imminent: days_until_expiry <= 90. Severity = critical if <= 30, high if <= 90.
- missing_clause: contract missing liability_cap_present=false or indemnification_present=false. Severity = critical.
- financial_anomaly: revenue drop > 20% YoY, OR audit_opinion is qualified or adverse, OR going_concern_note is true, OR debt_to_equity > 5. Severity = critical. One flag per anomaly.
- regulatory_risk: sub_processors_listed false, OR data residency conflict, OR breach_notification_period null. Severity = high per issue.
- certification_scope_gap: certificate scope does not cover vendor primary activity. Severity = medium.
- incomplete_submission: required fields null or unreadable. Severity = medium.

Return exactly this JSON structure:
{
  "packet_id": "",
  "document_name": "",
  "document_type": "",
  "classification_confidence": 0.0,
  "processing_mode": "single_pass",
  "extracted_fields": {},
  "flags": [],
  "completeness_check": {"expected_fields_present": true, "missing_expected_fields": []},
  "needs_verification_fields": [],
  "recommended_action": "",
  "action_reason": "",
  "summary": ""
}`;

  let messageContent;

  if (data.is_image) {
    messageContent = `Analyse the following document and return the JSON output as instructed.

Packet ID: ${data.packet_id}
Document filename: ${data.filename}
Vendor name: ${data.vendor_name}
Today's date: ${data.today}

IMPORTANT: You must set document_name in your JSON response to exactly this value: "${data.filename}" — do not change, shorten, or modify this filename in any way.

NOTE: This document is an image file (${data.media_type || 'image'}) named "${data.filename}". The image content could not be automatically extracted in this environment. Please:
1. Set document_name to exactly "${data.filename}"
2. Classify document_type as "insurance_certificate" based on the filename
3. Return empty extracted_fields with present: false and confidence: 0 for all insurance_certificate fields
4. Raise one incomplete_submission flag noting the image requires manual review
5. Set recommended_action to "review"
6. Note in summary that this is an image requiring manual extraction`;
  } else {
    messageContent = `Analyse the following document and return the JSON output as instructed.

Packet ID: ${data.packet_id}
Document filename: ${data.filename}
Vendor name: ${data.vendor_name}
Today's date: ${data.today}

IMPORTANT: You must set document_name in your JSON response to exactly this value: "${data.filename}" — do not change, shorten, or modify this filename in any way.

Document content:
${data.extracted_text}`;
  }

  const requestBody = {
    model: 'claude-sonnet-4-20250514',
    max_tokens: 2000,
    system: systemPrompt,
    messages: [{
      role: 'user',
      content: messageContent,
    }]
  };

  return {
    json: {
      ...data,
      claude_request_body: JSON.stringify(requestBody),
    }
  };
});

return results;
