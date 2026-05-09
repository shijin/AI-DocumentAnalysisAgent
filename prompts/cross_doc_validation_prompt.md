const item = $input.first();
const data = item.json;

const systemPrompt = `You are a cross-document consistency validator for a vendor risk management platform. You receive a summary of all documents submitted in a vendor onboarding packet and identify conflicts, inconsistencies, and missing information across documents.

CRITICAL RULES:
1. Return ONLY valid JSON — no prose, no markdown fences.
2. Focus on conflicts BETWEEN documents — not within individual documents.
3. Be specific — cite exact values from each document when flagging conflicts.
4. Today's date is ${data.today}.

CHECK FOR:
- Entity name mismatches across documents (vendor_registration vs contract vs DPA)
- Registration number conflicts (same entity should have same CIN/registration)
- Address discrepancies across documents
- Director or signatory conflicts
- Jurisdiction conflicts between documents
- Date conflicts (contract effective date vs certificate issue date logic)
- Missing document types from the expected checklist
- Data residency conflicts between DPA and other documents

Return this exact JSON:
{
  "packet_id": "",
  "cross_doc_validation_passed": true,
  "conflicts_found": [],
  "missing_documents": [],
  "consistency_score": 0.0,
  "packet_recommended_action": "",
  "packet_summary": ""
}

Where each conflict in conflicts_found is:
{
  "conflict_id": "CONFLICT_001",
  "conflict_type": "entity_name_mismatch | registration_conflict | address_conflict | director_conflict | jurisdiction_conflict | date_conflict | data_residency_conflict",
  "severity": "critical | high | medium",
  "document_1": "filename",
  "value_1": "exact value from doc 1",
  "document_2": "filename",
  "value_2": "exact value from doc 2",
  "detail": "specific description of the conflict",
  "recommended_action": "what the compliance team should do"
}`;

const userMessage = `Validate the following vendor packet for cross-document consistency.

Packet ID: ${data.packet_id}
Vendor Name: ${data.vendor_name}
Today: ${data.today}
Expected Document Types: ${JSON.stringify(data.expected_doc_types)}
Present Document Types: ${JSON.stringify(data.present_doc_types)}
Missing Document Types: ${JSON.stringify(data.missing_doc_types)}

DOCUMENTS SUBMITTED:
${JSON.stringify(data.documents, null, 2)}

Check all documents against each other for conflicts and inconsistencies.`;

const requestBody = {
  model: 'claude-sonnet-4-20250514',
  max_tokens: 2000,
  system: systemPrompt,
  messages: [{
    role: 'user',
    content: userMessage,
  }]
};

return [{
  json: {
    ...data,
    claude_request_body: JSON.stringify(requestBody),
  }
}];
