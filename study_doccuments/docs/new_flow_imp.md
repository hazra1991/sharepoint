[ Data Ingestion ] 
        ↓
[ LogDumpObject(s) ]
        ↓
[ Dispatcher ] → Looks at log.signature
        ↓
[ Analyzer ] (per type) → custom parsing, tagging, metadata extraction
        ↓
[ Optional Generic Preprocessing ]
        ↓
[ Output / Storage / ML ]