# NEI Nonroad Procedure

This procedure creates the NEI nonroad working table from CEIDARS inventory data.

## 1. Import EPA SCC File

Download the latest EPA SCC file from:

https://sor-scc-api.epa.gov/sccwebservices/sccsearch/

Import the SCC file into your own user schema. This documentation uses `ARMHASSAN.*` as the example schema/table prefix.

For this example, the imported table is:

```sql
ARMHASSAN.EPA_SCC_2026
```

One way to import the CSV is with:

```text
import_file2CEI.py
```

**NOTE:** The script is only one option. The SCC CSV can also be imported with SQL Developer, SQL*Loader, or another database import tool, as long as the final table has usable `SCC` and `DATA_CATEGORY` columns.

Verify the import:

```sql
select count(*)
from armhassan.EPA_SCC_2026
where data_category = 'Nonroad';
```

## 2. Run Pre-Checks

Run:

```sql
@nonroad_pre_checks.sql
```

This checks:

- EPA Nonroad SCCs missing from `CEI.SCC`
- numeric `CEI.EIC.EPA_AMS` values missing from `CEI.SCC`

Expected result for missing-SCC checks: **zero rows**.

If rows return, either update the SCC utility table or decide whether to move forward with the current CEIDARS reference tables.

## 3. Confirm Speciation And Reconciliation

Before creating the final nonroad extract, confirm that speciation and reconciliation have already been completed for the inventory year.

Run pre-speciation checks.

```sql
CEI.EMSyyyy
CEI.PROyyyy
```

Then run:

1. point speciation on `CEI.EMSyyyy`
2. area speciation on `CEI.EMSyyyy`
3. reconciliation on `CEI.EMSyyyy`
4. zero negative emissions in `CEI.EMSyyyy` as part of reconciliation cleanup

## 4. Create Nonroad Table

Create `NONROAD_yyyy` from the already-processed `CEI.PROyyyy` and `CEI.EMSyyyy` tables.

```sql
create table armhassan.nonroad_yyyy as
with eic_scc as (
    select
        a.*,
        case
            when regexp_like(epa_ams, '[A-Za-z]$')
                then regexp_replace(epa_ams, '[A-Za-z]$', '')
            else epa_ams
        end as epa_scc
    from cei.eic a
),
pro_ems_yyyy as (
    select *
    from cei.proyyyy a
    inner join cei.emsyyyy b
        using (co, ab, dis, facid, dev, proid)
)
select
    sub.*,
    c.epa_ams
from pro_ems_yyyy sub
inner join eic_scc c
    on sub.proid = c.eic
where c.epa_scc in (
    select scc
    from armhassan.EPA_SCC_2026
    where data_category = 'Nonroad'
);
```

## 5. Run Final QA

Run:

```sql
@nonroad_final_QA.sql
```

The final QA checks:

- OC1/OC2 exclusion
- duplicate emission keys
- missing pollutant lookup
- organic gas completeness
- PM completeness
- PM size order
- negative emissions
- agricultural burning missing acres

Expected result: **zero rows** for each issue check.

## 6. Fix Issues And Recreate If Needed

If final QA finds issues caused by live `CEI.EMSyyyy` data, fix the live inventory process first, then recreate `NONROAD_yyyy`.

Do not treat `NONROAD_yyyy` as final if `CEI.EMSyyyy` changes afterward.

## 7. Hand Off To Toxics

After QA passes, create or confirm the final handoff table in the `NEI` schema:

```sql
create table nei.nonroad_yyyy as
select *
from armhassan.nonroad_yyyy;
```

Toxics will copy the table from `NEI.NONROAD_yyyy`, run toxic speciation, QA the result, and return the final table name, usually in the `CTI` schema.

## 8. Expected Pollutants And Access Bridge

Compare against EPA expected pollutants if the expected pollutant file is available.

Create the final tables needed for the Microsoft Access Bridge Tool using:

```sql
@bridgetool_formaatter.sql
```

The formatter creates three bridge-ready tables:

- emissions
- emission process
- reporting period

The emissions table combines:

- CAPs from the QA-complete nonroad table
- HAPs/toxics from the toxics-speciated `CTI` table

Important formatting details:

- use `EPA_AMS` from `NONROAD_yyyy` as the SCC source
- split `EPA_AMS` into numeric `SourceClassificationCode` and suffix-based `EmissionsTypeCode`
- report toxics/HAPs in `LB`
- report CAPs/criteria in `TON`
- include methane/`CH4` in the toxics pollutant mapping if it is missing from the EPA pollutant crosswalk
- create process and reporting-period tables from the final emissions bridge table so the Bridge Tool hierarchy stays aligned

Export the three bridge tables to CSV, then import them into the Access Bridge Tool. During import, make sure **First Row Contains Field Names** is checked so the header row is not loaded as data.

One way to export the three Bridge Tool input CSVs is:

```text
export2csv_bridgetool_inputs.py
```

The Access Bridge Tool generates one final XML file for the nonroad submission.
