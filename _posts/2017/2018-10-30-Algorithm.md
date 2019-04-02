---
layout: "post"
title: "Algorithm"
date: "2018-10-30 10:30"
author: "Fang Zhou"
catalog: true
---
## Summary
- I try to modify the core algorithm today. I delete all of useless section(I guess).
  
```c++
double answer = 0;
	CString str;
	CEdit *pEdit = (CEdit*)GetDlgItem(IDC_EDIT_C);  
	pEdit->GetWindowText(str);
	int q;
	for (q = 0; q <= 10240; q++)
	{
		Data1_c[q] = (Data1[q] - (255 - m_txtCh1Position)) * 0.025 * GetVoltageDiv(Ch1.Voltage) * 2 ; 
		Data2_c[q] = (Data2[q] - (255 - m_txtCh2Position+1)) * 0.025 * GetVoltageDiv(Ch2.Voltage) * 1000 ;
	    answer +=  Data1_c[q]* Data2_c[q];
	}
	// Max & Min
	double max1 = Data1_c[0], max2 = Data2_c[0], min1 = 0, min2 = 0;
	
	for (q = 0; q <= 10240; q++)
	{
		if (max1 < Data1_c[q]) 
			max1 = Data1_c[q];
		if (max2 < Data2_c[q]) 
			max2 = Data2_c[q];
		if (min1 > Data1_c[q]) 
			min1 = Data1_c[q];
		if (min2 > Data2_c[q]) 
		{
			min2 = Data2_c[q];
		}
	}
	answer = answer/q;	

        m_voltage = max1;
	m_voltage2 = max2; 
	m_voltage3 = 2 * max2; 
	m_voltage4 = 2 * max1;
	
	return answer;
```

