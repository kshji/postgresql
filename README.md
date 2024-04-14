# Postgresql tools, info, sql, ...

## Postgresql uuidv7

UuidV7 and other uuid-based ones include, for example, the fact that the prefix is a timestamp, 
which makes indexing enjoyable and the timestamp does not need to be separately stored. 
It can be used for sorting just like SERIAL type keys.


  * [IETF](https://www.ietf.org/archive/id/draft-peabody-dispatch-new-uuid-format-04.html)
  * [Why uuidv7 is better ...](https://itnext.io/why-uuid7-is-better-than-uuid4-as-clustered-index-edb02bf70056)
  * [Good bye integers, welcome uuids](https://buildkite.com/blog/goodbye-integers-hello-uuids)
  * [Kvelakur C-language version using SSL Rand -uuidv4](https://gist.github.com/kvelakur/9069c9896577c3040030)
  * [My ksh shellscript to generate uuidV7](https://github.com/kshji/ksh/blob/master/Sh/uuidv7.sh)
  * [My uuid tools collection](ttps://github.com/kshji/uuid/)

### Extension

  * [PG_uuidv7 extension](https://pgxn.org/dist/pg_uuidv7/)
 
### Functions

  * [PG uuidv7 functions](https://gist.github.com/kjmph/5bd772b2c2df145aa645b837da7eca74)

#### My select from previous functions

##### Generate uuidv7
```sql
CREATE OR REPLACE FUNCTION public.uuid_generate_v7()
RETURNS uuid
AS $$
  -- use random v4 uuid as starting point (which has the same variant we need)
  -- then overlay timestamp
  -- then set version 7 by flipping the 2 and 1 bit in the version 4 string
select encode(
    set_bit(
      set_bit(
        overlay(uuid_send(gen_random_uuid())
                placing substring(int8send(floor(extract(epoch from clock_timestamp()) * 1000)::bigint) from 3)
                from 1 for 6
        ),
        52, 1
      ),
      53, 1
    ),
    'hex')::uuid;
$$
language SQL
volatile;
```

##### Get timestamp from uuidv7
```sql
CREATE OR REPLACE FUNCTION public.timestamp_from_uuid_v7(_uuid uuid)
RETURNS timestamp without time zone
LANGUAGE sql
IMMUTABLE PARALLEL SAFE STRICT LEAKPROOF
AS $$
  SELECT to_timestamp(('x0000' || substr(_uuid::text, 1, 8) || substr(_uuid::text, 10, 4))::bit(64)::bigint::numeric / 1000);
$$
language SQL
volatile;
;

```

