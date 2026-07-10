
## Import and object construction

1. Import each library separately in sparse format.
2. Preserve filtered and raw matrices.
3. Add sample, condition, replicate, and batch metadata.
4. Prefix barcodes to create globally unique cell identifiers.
5. verify feature compatibility across libraries.
6. Save per-library objects.
7. Combine filtered objects for convenience, but defer integration until after library-specific QC.