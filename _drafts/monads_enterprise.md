---
layout: post
title: monads fix enterprise programming
---

# {{ page.title }}



    private static String buildOrderBy(
            final IOrmTypeInfo typeInfo,
            final QueryOptions queryOptions)
    {
        verify(null != queryOptions, "");
        verify(null != queryOptions.getSortKeys(), "");

        List<String> orders = new ArrayList<String>()
        {{
            for(SortKey sortKey : queryOptions.getSortKeys())
            {
                verify(null != sortKey, "sort key is null");

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

        orders.add(IOrmObject.ATTR_ID); // always order by ormid - orm impl detail

        return format("ORDER BY %s %s", join(orders, ","));
    }




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
