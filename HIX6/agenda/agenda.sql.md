# @Source agenda.sql

```sql
/** 
 *   @Source test.sql
 *   @Description 
 */

declare @start date;
declare @eind date;
set @start = '20190101';
set @eind = '20191231';

if object_id('tempdb..#agenda') is not null drop table #agenda;
with cte_specialist as (
    select
    t00.artscode,
    ltrim(case when isnull(achternaam,'') = '' then '' else ltrim(achternaam)  COLLATE database_default + ' ' end
       +
       case when isnull(achternaam,'') <> '' and isnull(meisjesnaa,'') <> '' then ' - ' else '' end
       +
       case when isnull(meisjesnaa,'') = '' then '' else ltrim(meisjesnaa) + ' ' end
       +
       case when isnull(voorvoegm,'') = '' then '' else lower(voorvoegm)  end
        +
       case when isnull(voorvoega,'') = '' then '' else lower(voorvoega) + ' ' end
       +
       case when isnull(voorletter,'') = '' then '' else voorletter end) specialist_naam,
    case
    when isnull(t00.Specialism, '') = ''
    then null
    else t00.Specialism end specialisme_code
    from CSZISLIB_ARTS t00
)
select
   t00.afspraaknr afspraaknummer,
   t00.patientnr patientnummer,
   format(t00.datum+t00.tijd, 'yyyy-MM-dd HH:mm') [datumtijd afspraak],
   t00.agenda [agenda code],
   t30.omschr [agenda oms],
   t00.subagenda [subagenda code],
   t20.naam [subagenda naam],
   t20.omschr as [subagenda oms],
   t00.code as [type afspraak code],
   t10.omschr [type afspraak oms],
   t00.constype as consulttype,
   case t00.constype
       when 'E' then 'Eerste'
       when '*' then 'Geen'
       when 'H' then 'Herhaling'
       when 'I' then 'Intercollegiaal'
       when 'K' then 'Keuring'
       when 'T' then 'Telefonisch'
       when 'A' then 'Traumatologisch'
       when 'V' then 'Verrichting'
       when 'M' then 'E-Mail'
       when 'S' then 'Screen to screen'
       when 'B' then 'Medebehandeling'
       else 'Onbekend' end [constype oms],
   t00.uitvoerder as uitvoerder_code,
   t50.specialist_naam as specialist_naam_uitv,
   t50.specialisme_code as specialisme_code_uitvoerder,
   t00.aanvrager as aanvrager_code,
   t40.specialist_naam as specialist_naam_aanv,
   t40.specialisme_code as specialisme_code_aanvrager,
   t00.episode dbc_nummer
into #agenda
from [dbo].agenda_afspraak t00
 left join [dbo].agenda_afspcode t10
  on t00.agenda = t10.agenda
   and t00.code = t10.code
   and t10.vervallen = 0
 left join [dbo].agenda_subagend t20
  on t00.subagenda = t20.subagenda
 left join [dbo].agenda_agenda t30
  on t00.agenda = t30.agenda
 left join cte_specialist t40
  on t00.aanvrager = t40.artscode
 left join cte_specialist t50
  on t00.uitvoerder = t50.artscode
where t00.AGENDA in ('..', '..')
   and t00.CONSTYPE = '..' -- eerste consult bijvoorbeeld
   and t00.code = '' --afspraakcode
   and t00.datum between @start and @eind
   and t00.voldaan <> 'N' -- alleen doorgegaan

select
   *
 from #agenda t00
 order by [datumtijd afspraak]
 ```