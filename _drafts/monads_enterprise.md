---
layout: post
title: monads fix enterprise programming
---

# {{ page.title }}


here is some code in production at work. We have some tables in our product, in which we put lists of objects. Each row to an object, each column to an attribute on the object. Think of an object like this:

    {
      name: 'Dustin',
      age: 27,
      gender: Genders.Male
    }

This code generates the "order by" part of the SQL query powering the table. (Don't look at this too long. Just note that it's function which builds a string from some arguments. It has no side effects.)

    private static String buildOrderBy(
            final IOrmTypeInfo typeInfo,
            final QueryOptions queryOptions)
    {
        List<String> orders = new ArrayList<String>()
        {{
            for(SortKey sortKey : queryOptions.getSortKeys())
            {
                String fieldName = sortKey.getColumnName();
                IOrmTypeFieldInfo field = typeInfo.getTypeFieldInfo(fieldName);

                String order = format("%s %s",
                        sortKey.getColumnName(),
                        sortKey.getOrder() == SortOrder.DESCENDING ? "DESC" : "");
                add(order);
            }
        }};

        return format("ORDER BY %s %s", join(orders, ","));
    }

So, this is great, until you add some features. The new guy is working on the frontend widgets for the tables, so sometimes this gets called with bad data and we want to catch it during development before QA and release, so we add a verify so the frontend guy gets a nice error message during development when his code is wrong. We also now have attributes which are a list:

    {
      name: 'Dustin',
      age: 27,
      gender: Genders.Male,
      wives: ['Alice', 'Beth', 'Cindy']
    }

but we don't want to allow you to sort these columns because it doesn't make sense.

    private static String buildOrderBy(
            final IOrmTypeInfo typeInfo,
            final QueryOptions queryOptions)
    {
        List<String> orders = new ArrayList<String>()
        {{
            for(SortKey sortKey : queryOptions.getSortKeys())
            {
                String fieldName = sortKey.getColumnName();
                IOrmTypeFieldInfo field = typeInfo.getTypeFieldInfo(fieldName);

                verify(null != field, "Sort Key Field %s does not exist on type %s.", fieldName, typeInfo.getName());
                verify(!field.isRepeating(), "Repeating values not permitted in sort: %s", fieldName);

                String order = format("%s %s",
                        sortKey.getColumnName(),
                        sortKey.getOrder() == SortOrder.DESCENDING ? "DESC" : "");
                add(order);
            }
        }};

        return format("ORDER BY %s %s", join(orders, ","));
    }

Now we have a problem. `verify` throws an exception, which by design, will crash the page with an error message. It's an internal message, designed to help catch and diagnose bugs. But we're trying to ship, and our client side code still allows you to sort on list columns, and now that crashes the page for our clients. We want to signal a warning instead, but continue with execution instead of throwing.

Exceptions can't do this.

The easiest thing to do here is have some sort of state. We could add a global, like a logger, that logs warnings, and later we will report the warnings to the client. But



















    private static void appendOrderBy(StringBuilder sb, IOrmTypeInfo typeInfo, String viewPrefix, QueryOptions queryOptions, boolean skipIndex)
    {


        sb.append(" ORDER BY ");

        if(null != queryOptions && null != queryOptions.getSortKeys())
        {
            for(SortKey sortKey : queryOptions.getSortKeys())
            {
                DebugUtil.verify(null != sortKey, "sort key is null");

                String fieldName = sortKey.getColumnName();

                IOrmTypeFieldInfo field = typeInfo.getTypeFieldInfo(fieldName);
                DebugUtil.verify(null != field, "Sort Key Field %s does not exist on type %s.", fieldName, typeInfo.getName());
                DebugUtil.verify(!field.isRepeating(), "Repeating values not permitted in sort: %s", fieldName);

                boolean useUpper = String.class.isAssignableFrom(field.getJavaDataType());
                if(useUpper){
                    sb.append("UPPER(");
                }
                appendPrefix(sb, viewPrefix);
                sb.append(sortKey.getColumnName());
                if(useUpper){
                    sb.append(")");
                }

                if(sortKey.getOrder() == SortOrder.DESCENDING)
                {
                    sb.append(" DESC");
                }

                sb.append(", ");
            }
        }

        appendPrefix(sb, viewPrefix);
        sb.append(IOrmObject.ATTR_ID);

        if(!skipIndex && typeInfo.hasRepeatingValues())
        {
            // add the index to the order by so that the repeating values come back in the right order
            sb.append(", ");

            appendPrefix(sb, viewPrefix);
            sb.append(IOrmObject.ATTR_I_INDEX);
        }

    }
