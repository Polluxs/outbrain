I'm uploading a HAR file from visiting [website] with [consent choice: accepted/rejected/no interaction]. Please analyze for privacy and tracking concerns, specifically:

## 1. Outbrain-Specific Analysis
- List all Outbrain domains contacted (including subdomains)
- Decode and analyze the token parameter structure
- Compare behavior against Desmet's findings (especially domains: amplify-imp.outbrain.com, b1sync.outbrain.com)
- Identify data sent despite consent status
- Timeline: What happens before/during/after consent check?

## 2. Parameter Deep Dive
Focus on the token structure like: 3127321feab44467e66824f02af29064_230797_1757325875926_1
- What does each segment represent?
- Which parameters persist across requests?
- Any parameters under 12 characters? (Desmet missed these)
- Encoded/obfuscated data?

## 3. Privacy Violations Check
- Data collection before consent determination
- Browser fingerprinting attempts (canvas, WebGL, fonts)
- Third-party syncing (especially cross-domain)
- Persistent identifiers
- Storage mechanisms (cookies, localStorage)

## 4. Technical Patterns
- Request sequence and timing
- Which requests are conditional on consent?
- Differences between first visit vs. return visit
- Server response variations

## 5. Comparison Points
- How does this align/conflict with Desmet's "vroegtijdig gestopt" claim?
- Evidence of "contextueel" vs. tracking behavior?
- What data flows regardless of consent?

## 6. Red Flags
Highlight any:
- Unexpected data collection
- Consent bypass mechanisms
- Hidden tracking pixels
- Encoded user identifiers
- Cross-site correlation attempts

Please present findings in order of privacy impact severity.
