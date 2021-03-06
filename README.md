#R Assignment



##Load libraries

library(ggplot2)

library(tidyverse)



##Read files

fang_et_al_genotypes <- read.table("C:/Users/kenziewishman/Desktop/BCB546-Spring2021/assignments/R_Assignment/fang_et_al_genotypes.txt", header = T)

snp_position <- read.table("C:/Users/kenziewishman/Desktop/BCB546-Spring2021/assignments/R_Assignment/snp_position.txt", sep = "\t", header = T)



##Data inspection

str(fang_et_al_genotypes)
nrow(fang_et_al_genotypes)
ncol(fang_et_al_genotypes)
head(fang_et_al_genotypes)
tail(fang_et_al_genotypes)
sapply(fang_et_al_genotypes, class)

###fang_et_al_genotypes: 2782 rows, 986 columns



str(snp_position)
nrow(snp_position)
ncol(snp_position)
head(snp_position)
tail(snp_position)
sapply(fang_et_al_genotypes, class)

###snp_position: 983 rows, 15 columns



##Separate maize and teosinte SNPs

maize_genotypes <- filter(fang_et_al_genotypes, Group == "ZMMLR" | Group == "ZMMIL" | Group == "ZMMMR")

teosinte_genotypes <- filter(fang_et_al_genotypes, Group == “ZMPBA” | Group == “ZMPIL” | Group == “ZMPJA”)



##Transpose maize and teosinte files

maize_transposed <- data.frame(t(maize_genotypes))

teosinte_transposed <- data.frame(t(teosinte_genotypes))



##Combine teosinte/maize SNPs with SNPs Position

maize_combined <- merge(snp_position, maize_transposed, by = 0)

teosinte_combined<- merge(snp_position, teosinte_transposed, by = 0)



##Separate multiple and unknown into dfs

maize_unknown <- subset.data.frame(maize_combined, maize_combined$Chromosome == "unknown")

teosinte_unknown <- subset.data.frame(teosinte_combined, teosinte_combined$Chromosome == "unknown")

maize_multiple <- subset.data.frame(maize_combined, maize_combined$Chromosome == "multiple")

teosinte_multiple<- subset.data.frame(teosinte_combined, teosinte_combined$Chromosome == "multiple")



##Make position and chromosome columns numeric

teosinte_combined$Chromosome <- as.numeric(teosinte_combined$Chromosome)

teosinte_combined$Position <- as.numeric(teosinte_combined$Position)

maize_combined$Chromosome <- as.numeric(maize_combined$Chromosome)

maize_combined$Position <- as.numeric(maize_combined$Position)



##Sort by chromosome and position

maize_increasing <- maize_combined[order(maize_combined$Chromosome, maize_combined$Position),]

teosinte_increasing <- teosinte_combined[order(teosinte_combined$Chromosome, teosinte_combined$Position),]



##Remove NA

maize_increasing <- maize_increasing %>% filter(!is.na(maize_increasing$Chromosome))

teosinte_increasing <- teosinte_increasing %>% filter(!is.na(teosinte_increasing$Chromosome))



##Create naming vector

maize_names <- c("maize1","maize2","maize3","maize4","maize5","maize6","maize7","maize8","maize9","maize10")

teosinte_names <- c("teosinte1","teosinte2","teosinte3","teosinte4","teosinte5","teosinte6","teosinte7","teosinte8","teosinte9","teosinte10")



##Save into files subsetting by chromosome number

for (i in 1:length(maize_increasing$Chromosome)) {write.table(subset.data.frame(maize_increasing, maize_increasing$Chromosome == i),
file = paste0("asc",maize_names[i], ".txt"), sep = "\t", row.names = F, quote = F)}

for (i in 1:length(teosinte_increasing$Chromosome)) {write.table(subset.data.frame(teosinte_increasing, teosinte_increasing$Chromosome == i), file = paste("asc",teosinte_names[i], ".txt"), sep = "\t", row.names = F, quote = F)}



##Decreasing SNPs

maize_decreasing <- maize_combined[order(maize_combined$Chromosome, -maize_combined$Position),]

teosinte_decreasing <- teosinte_combined[order(teosinte_combined$Chromosome, -teosinte_combined$Position),]



##Replace "?" with "-" in SNP information

maize_decreasing <- as_tibble(lapply(maize_decreasing, gsub, pattern="?", replacement="-", fixed=T))

teosinte_decreasing <- as_tibble(lapply(teosinte_decreasing, gsub, pattern="?", replacement= "-", fixed=T))



##Subset by chromosome number

for (i in 1:length(maize_decreasing$Chromosome)) {write.table(subset.data.frame(maize_decreasing, maize_decreasing$Chromosome == i),
file = paste("desc", maize_names[i], ".txt"), sep = "\t", row.names = F, quote = F)}

for (i in 1:length(teosinte_decreasing$Chromosome)) {write.table(subset.data.frame(teosinte_decreasing, teosinte_decreasing%Chromosome == i), file = paste("desc", teosinte_names[i], ".txt"), sep = "\t", row.names = F, quote = F)}



##Data Visualization



###Reshape data

teosinte_long <- teosinte_combined %>% pivot_longer(cols = starts_with("X"), names_to = NULL, names_prefix = "X", values_to = "SNP",)

maize_long <- maize_combined %>% pivot_longer(cols = starts_with("X"), names_to = NULL, names_prefix = "X", values_to = "SNP",)



##Change Chromosome numbers to factors

maize_combined$Chromosome <- factor(maize_combined$Chromosome, levels = c(1,2,3,4,5,6,7,8,9,10))

teosinte_combined$Chromosome <- factor(teosinte_combined$Chromosome, levels = c(1,2,3,4,5,6,7,8,9,10))



##Split SNP column 

teosinte_split <- teosinte_long %>% separate(SNP, c("SNP1", "SNP2"), remove = F)

maize_split <- maize_long %>% separate(SNP, c("SNP1","SNP2"), remove = F)



##Class as heterozygote based on whether the two snp columns are the same

teosinte_split$hetero <- ifelse(teosinte_split$SNP1 == teosinte_split$SNP2, "N", "Y")

maize_split$hetero <- ifelse(maize_split$SNP1 == maize_split$SNP2, "N", "Y")



##Replace missing values

maize_split <- maize_split %>% mutate(hetero = replace(hetero, SNP1 == "", "missing"))

teosinte_split <- teosinte_split %>% mutate(hetero = replace(hetero, SNP1 == "", "missing"))



##SNP number per chromosome

maize_SNP_number <- ggplot(maize_combined) + geom_bar(aes(x = Chromosome, fill = Chromosome))

ggsave("maize_SNP_locations.png", plot = maize_SNP_number)

teosinte_SNP_number <- ggplot(teosinte_combined) + geom_bar(aes(x = Chromosome, fill = Chromosome)) 

ggsave("teosinte_SNP_locations.png", plot = teosinte_SNP_number)



##Density plots

maize_density <- ggplot(maize_combined) + geom_density(aes(x=Position, fill = Chromosome)) 

ggsave("maize_SNP_density.png", plot = maize_density)

teosinte_density <- ggplot(teosinte_combined) + geom_density(aes(x=Position, fill = Chromosome)) 

ggsave("teosinte_SNP_density.png", plot = teosinte_density)



maize_density <- ggplot(maize_combined) + geom_density(aes(x=Position, fill = Chromosome)) + facet_wrap(~Chromosome)

ggsave("maize_SNP_density_f.png", plot = maize_density)

teosinte_all_density <- ggplot(teosinte_combined) + geom_density(aes(x=Position, fill = Chromosome)) + facet_wrap(~Chromosome)

ggsave("teosinte_SNP_density_f.png", plot = teosinte_density)



##Missing data and heterozygosity plots

maize_hetero_plot <- ggplot(maize_split, aes(x=hetero, fill = Chromosome)) + geom_bar() 

ggsave("maize_heterozygosity.png", plot = maize_hetero_plot)

teosinte_hetero_plot <- ggplot(teosinte_split, aes(x=hetero, fill = Chromosome)) + geom_bar()

ggsave("teosinte_heterozygosity.png", plot = teosinte_hetero_plot)



##Dot plot of position on Chromosomes

maize_dot <- ggplot(maize_combined, aes(x=Position, y=Chromosome, fill= Chromosome)) + geom_point() 

ggsave("SNP_Position_dot_plot_maize.png", plot = maize_dot)

teosinte_dot <- ggplot(teosinte_combined, aes(x=Position, y=Chromosome, fill= Chromosome)) + geom_point() 

ggsave("SNP_Position_dot_plot_teosinte.png", plot = teosinte_dot)
