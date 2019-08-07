# [Toyota Profits, Imports Pull Ahead, Tariffs Just Around the Corner](https://panjiva.com/research/toyota-profits-imports-pull-ahead-tariffs-just-around-the-corner/28476)

The above charts were created on Aug. 6, 2019 using the following query: At a simple level, the query uses a base U.S. import search, selecting the time, origin, bill of lading for shipment count, volume and HS code. A cross apply is used to deaggregate the HS codes, relying on the union to clear any duplicates. Three more years of Panjiva data are added using the same query, and then used as a subquery. 

The main query selects from the subquery, aggregating on shipment count and volume.

Two joins are used to allow a search on the CIQ id for Toyota, resolving through the cross reference table and CIQ ultimate parents table. The final charts were produced using pivot tables in Excel, adding in S&P Market Intelligence data where necessary.

```sql
SELECT conPanjivaId, shpPanjivaId, yr, mo, origin,
  count(DISTINCT billOfLadingNumber), sum(volumeTEU), hs
FROM(
  SELECT conPanjivaId, shpPanjivaId, datepart(year, arrivalDate) yr,
    datepart(month, arrivalDate) mo, shpmtOrigin origin, billOfLadingNumber,
    volumeTEU, left(trim(replace(value, '.', '')), 4) hs
    --select all fields needed
  FROM XFL_PANJIVA.dbo.panjivaUSImport2016 im
  	--from us import data
    JOIN XFL_PANJIVA.dbo.panjivaUSImpHSCode2016 hs
    --join hs code table on record ids
    ON hs.panjivaRecordId = im.panjivaRecordId
  --cross apply the hs codes to each record
  CROSS APPLY string_split(replace(hs.hscode, '.', ''),';')
  UNION --add 2017
  SELECT conPanjivaId, shpPanjivaId, datepart(year, arrivalDate) yr,
    datepart(month, arrivalDate) mo, shpmtOrigin origin, billOfLadingNumber,
    volumeTEU, left(trim(replace(value, '.', '')), 4) hs
  FROM XFL_PANJIVA.dbo.panjivaUSImport2017 im
    JOIN XFL_PANJIVA.dbo.panjivaUSImpHSCode2017 hs
    ON hs.panjivaRecordId = im.panjivaRecordId
  CROSS APPLY string_split(replace(hs.hscode, '.', ''),';')
  UNION --add 2018
  SELECT conPanjivaId, shpPanjivaId, datepart(year, arrivalDate) yr,
    datepart(month, arrivalDate) mo, shpmtOrigin origin, billOfLadingNumber,
    volumeTEU, left(trim(replace(value, '.', '')), 4) hs
  FROM XFL_PANJIVA.dbo.panjivaUSImport2018 im
    JOIN XFL_PANJIVA.dbo.panjivaUSImpHSCode2018 hs
    ON hs.panjivaRecordId = im.panjivaRecordId
  CROSS APPLY string_split(replace(hs.hscode, '.', ''),';')
  UNION --add 2019
  SELECT conPanjivaId, shpPanjivaId, datepart(year, arrivalDate) yr,
    datepart(month, arrivalDate) mo, shpmtOrigin origin, billOfLadingNumber,
    volumeTEU, left(trim(replace(value, '.', '')), 4) hs
  FROM XFL_PANJIVA.dbo.panjivaUSImport2019 im
    JOIN XFL_PANJIVA.dbo.panjivaUSImpHSCode2019 hs
    ON hs.panjivaRecordId = im.panjivaRecordId
  CROSS APPLY string_split(replace(hs.hscode, '.', ''),';')
  ) im
JOIN (
  SELECT c.companyID as ParentID, c.companyName as Parent, ccr.identifiervalue
  --select company information
  FROM xfl_ciq.dbo.ciqCompany c
  --source from the ciq company table
  JOIN xfl_ciq.dbo.ciqCompanyUltimateParent cup
    ON cup.ultimateParentCompanyId = c.companyId
    --join to the ciq ultimate parents table
  JOIN xfl_panjiva.dbo.panjivaCompanyCrossRef ccr
    ON ccr.companyId = cup.companyId
    --join to panjiva cross reference table
  GROUP BY c.companyId, c.companyName, ccr.identifiervalue
  --search on toyota ciqid
  ) coc ON im.conPanjivaId = coc.identifiervalue 
    AND coc.ParentID = 319676
  --link to consingee
JOIN (
  SELECT c.companyID as ParentID, c.companyName as Parent, ccr.identifiervalue
  FROM xfl_ciq.dbo.ciqCompany c
  JOIN xfl_ciq.dbo.ciqCompanyUltimateParent cup
    ON cup.ultimateParentCompanyId = c.companyId
  JOIN xfl_panjiva.dbo.panjivaCompanyCrossRef ccr
    ON ccr.companyId = cup.companyId
  GROUP BY c.companyId, c.companyName, ccr.identifiervalue
  --search on toyota ciqid
  ) csh ON im.shpPanjivaId = csh.identifiervalue 
    AND csh.ParentID = 319676
  --link to shipper
GROUP BY conPanjivaId, shpPanjivaId, yr, mo, origin, hs
```
