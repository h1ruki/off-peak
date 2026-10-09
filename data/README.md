# Data

Off-Peak uses TfL's daily **Station Footfall** data. The seven CSVs in `data/raw/` are versioned in this repo as the pinned snapshot audited in Phase 0. Their latest observation is 4 July 2026. Other raw files and downloaded archives are not committed.

- **Source:** [TfL network demand data](https://tfl.gov.uk/corporate/publications-and-reports/network-demand-data), Station Footfall documents
- **Snapshot:** downloaded 9 Oct 2026, covering 1 Jan 2019 to 4 Jul 2026
- **Licence:** Powered by TfL Open Data. TfL's terms are based on the Open Government Licence v2.0 ([terms](https://tfl.gov.uk/corporate/terms-and-conditions/transport-data-service)).
- **Also used:** [UK bank holidays](https://www.gov.uk/bank-holidays.json), England and Wales

## Files in the pinned snapshot

| File | Dates | Rows | Bytes | SHA-256 |
| --- | --- | --- | --- | --- |
| StationFootfall_2019.csv | 2019-01-01 to 2019-12-31 | 151,639 | 6,318,172 | `716984ede6aa2ce6c0dbd877cf228fc1894cf136cbc17534ca039de0d664a0eb` |
| StationFootfall_2020.csv | 2020-01-01 to 2020-12-31 | 150,089 | 6,085,851 | `bf4bb032cf1dac87d7989a20bd19984c5c0f7f1a868c3c5d6de49ed9c2222ab2` |
| StationFootfall_2021.csv | 2021-01-01 to 2021-12-31 | 154,167 | 6,303,035 | `6d8007a1c49cf8042ac43fd45054d9b0c854355932622fc2980a9ca564597777` |
| StationFootfall_2022.csv | 2022-01-01 to 2022-12-31 | 155,130 | 6,409,862 | `892663411d703558a16d72ebf08e18a652d88a4343dc56f06dd7e7f69cf564b2` |
| StationFootfall_2023.csv | 2023-01-01 to 2023-12-31 | 156,491 | 6,500,306 | `2e1714452dc5e83819d7bf287e21333856391024bb25f636813b978f5f434355` |
| StationFootfall_2024.csv | 2024-01-01 to 2024-12-31 | 156,995 | 6,525,587 | `7642ac542ba1925a673d348b74619d6f2973f00b11531d8ff773251cc0bbb8e4` |
| stationfootfall-2025-2026.csv | 2025-01-01 to 2026-07-04 | 235,636 | 9,809,630 | `6bf838b3b815bdc2fb56497494b6f60ca202d1b949f6aaaf25ce78da3fac3211` |

Rows exclude the header line.

## Verify your copy (Windows PowerShell)

```powershell
Get-FileHash .\data\raw\*.csv -Algorithm SHA256 | Format-Table -AutoSize
```

Every hash should match the table. If TfL has published newer files, keep this snapshot for v1 so the report and the validation results stay reproducible. Replacing it requires a new audit, new checksums and an update to this document before the files are committed.
