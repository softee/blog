---
layout: post
title:  "solve python rounding problem"
date:   2018-04-05 08:00:30 +0800
categories: python
---
-   python version:2.7.9
-   problem decription: when use rounding, the results is not inaccurate, for example：

        >>> round(2.655,2)
        2.65
        >>> round(2.485,2)
        2.48
        >>> round(1.115,2)
        1.11
-   These can't work:

        >>> from decimal import Decimal
        >>> '{:.2f}'.format(Decimal('2.485'))  
        '2.48'
        >>> '{:.2f}'.format(2.675)  
        '2.67'  
-   I solve it like this:

        def round_up(val,n):
            """
            :function:replace the bulid-in funciton round, make it more accurate when round.    
            :param val: input value,string or numerical type.
            :param n: decimal place,integer,bigger than zero.
            :return: rounding results，string
            """
            n = int(n)
            f = '{:.%sf}'%n
            return f.format(round(float(val) * pow(10,n)) / pow(10.0,n))

        if __name__ == '__main__':  
            print round_up(1.115,2) # test value:2.4449,2.655,2.665,1.115